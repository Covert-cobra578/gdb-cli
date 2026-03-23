# Async Load Optimization

**Date**: 2026-03-23
**Status**: Draft
**Scope**: `launcher.py`, `gdb_rpc_server.py`, `cli.py`

## Problem

`gdb-cli load` is synchronous — it blocks until GDB finishes loading binary symbols and core file mappings. For large targets (e.g., 8GB binary + 63GB core), this takes 300-600 seconds. The CLI's `_wait_for_socket` (hardcoded 300s timeout) fires before GDB finishes, causing load failure.

Root cause: the RPC Server socket is created **after** `core-file` completes. The CLI polls for the socket file and times out waiting for the slow `core-file` command.

## Solution

Reorder GDB `-ex` commands so that the RPC Server starts **before** loading binary/core. This makes the socket available immediately, and the `load` command returns asynchronously with `"status": "loading"`. Callers poll `gdb-cli status` until the session reaches `"ready"`.

## Design

### 1. GDB Command Reorder (`launcher.py`)

**Before:**
```
set pagination off / set print elements 0 / set confirm off
[sysroot / solib-prefix / source_dir]
file {binary}
core-file {core}
python import os; os.environ[...] = ...   (x4)
source gdb_rpc_server.py
python start_server(...)       # socket created here (too late)
```

**After:**
```
set pagination off / set print elements 0 / set confirm off
[sysroot / solib-prefix / source_dir]
python import os; os.environ[...] = ...   (x4)
source gdb_rpc_server.py
python start_server(...)       # socket created here (early)
file {binary}
core-file {core}
python _gdb_rpc_server.set_ready()   # mark session ready
```

**Why this works:** `start_server()` spawns a background `accept_thread` (threading.Thread, daemon=True). Even while GDB's main thread is blocked on `core-file`, the accept thread can handle socket connections. During loading, only `ping` and `status` are allowed — these use a lightweight handler on `GDBRPCServer` itself that does **not** call any GDB API, so they are safe to run from the accept thread while the main thread is blocked.

**Parameter plumbing for timeout:** `_start_gdb_process` currently takes `(gdb_args, session)`. Add a `timeout` parameter so the user's `--timeout` value flows through: `launch_core()` → `_start_gdb_process(gdb_args, session, timeout)` → `_wait_for_socket(..., timeout=float(timeout))`.

### 2. Server State Machine (`gdb_rpc_server.py`)

Add `_state` field to `GDBRPCServer`:

```python
class GDBRPCServer:
    def __init__(self, ...):
        ...
        self._state = "loading"        # "loading" | "ready"
        self._loading_start = time.time()

    def set_ready(self):
        """Called after core-file completes. Thread-safe via CPython GIL."""
        self._state = "ready"
```

**Thread safety note:** `_state` is a simple string attribute read/written by different threads (accept thread reads, main thread writes via `set_ready()`). This is safe under CPython's GIL — simple attribute assignment is atomic. GDB's embedded Python uses CPython, so the GIL guarantee holds.

**Loading guard in `_dispatch` method (before handler lookup):**

```python
def _dispatch(self, request):
    cmd = request.get("cmd")
    if not cmd:
        raise ValueError("Missing 'cmd' in request")

    # Loading guard: only ping/status allowed during loading
    if self._state == "loading" and cmd not in ("ping", "status"):
        elapsed = time.time() - self._loading_start
        raise ValueError(f"Session is loading ({elapsed:.0f}s elapsed)")

    # For "status" during loading, use the lightweight built-in handler
    # (the handlers.handle_status calls GDB API which is unsafe during loading)
    if cmd == "status" and self._state == "loading":
        return self._handle_loading_status()

    handler = self._handlers.get(cmd)
    if not handler:
        raise ValueError(f"Unknown command: {cmd}")
    ...
```

This guard is placed **before** handler dispatch, ensuring `handlers.handle_status` (which calls `gdb.selected_inferior()` etc.) is never invoked during loading. Instead, the lightweight `_handle_loading_status` is used.

**Lightweight status handler (no GDB API calls):**
```python
def _handle_loading_status(self):
    """Status during loading — no GDB API calls, safe from accept thread."""
    return {
        "state": "loading",
        "elapsed": time.time() - self._loading_start,
        "session_meta": self.session_meta,
    }

def _handle_ping(self, **kwargs):
    """Ping — always safe, no GDB API calls."""
    return {"pong": True, "time": time.time(), "state": self._state}
```

After `set_ready()`, the normal `handlers.handle_status` is used (dispatched via `self._handlers["status"]`).

**Global variable for `set_ready()` access:**

GDB's `source` command loads the script into GDB's Python `__main__` namespace. But `global _gdb_rpc_server` inside `start_server()` sets the variable in the **module's** namespace (where `start_server` is defined), not in `__main__`. The subsequent `-ex python _gdb_rpc_server.set_ready()` runs in `__main__` and would raise `NameError`.

**Fix:** Explicitly inject into `__main__`:

```python
def start_server(sock_path, session_meta, heartbeat_timeout=600):
    server = GDBRPCServer(sock_path, session_meta, heartbeat_timeout)
    server.start()
    # Inject into __main__ so subsequent `-ex python` commands can access it
    import __main__
    __main__._gdb_rpc_server = server
    return server
```

Launcher appends: `python _gdb_rpc_server.set_ready()`

This works because both `source gdb_rpc_server.py` and `-ex python ...` share GDB's `__main__` namespace.

### 3. CLI Changes (`cli.py`)

**`load` command:** Returns immediately with `"status": "loading"` instead of `"started"`:
```json
{
  "session_id": "abc12345",
  "mode": "core",
  "binary": "/path/to/observer",
  "core": "/path/to/core",
  "gdb_pid": 12345,
  "status": "loading"
}
```

**Breaking change note:** The `status` field changes from `"started"` to `"loading"`. The only caller is AI agents (Claude Code), and the usage docs will be updated accordingly. No backward compatibility shim needed.

**`status` command fallback:** When socket connection fails, check if GDB process is still alive:

```python
@main.command()
@click.option("--session", "-s", required=True, help="会话 ID")
def status(session: str):
    try:
        with get_client(session) as client:
            result = client.status()
            result["session_id"] = session
            print_json(result)
    except GDBClientError:
        # Fallback: check if GDB process is alive
        meta = get_session(session)
        if meta and meta.gdb_pid:
            try:
                os.kill(meta.gdb_pid, 0)
                # Process alive but socket not responding
                print_json({
                    "session_id": session,
                    "state": "loading",
                    "message": "GDB process alive, not yet responding"
                })
            except OSError:
                print_error("Session dead", f"GDB process {meta.gdb_pid} no longer exists")
                raise click.exceptions.Exit(1)
        else:
            print_error("Session not found", session)
            raise click.exceptions.Exit(1)
```

### 4. Timeout Fix (`launcher.py`)

Add `timeout` parameter to `_start_gdb_process`:

```python
def _start_gdb_process(gdb_args, session, timeout=600):
    ...
    _wait_for_socket(Path(session.sock_path), timeout=float(timeout))
    ...
```

Call site in `launch_core`:
```python
_start_gdb_process(gdb_args, session, timeout=timeout)
```

In practice this wait returns in <1 second (socket is created before `file`/`core-file`), but the timeout acts as a safety net.

### 5. GDB stderr Logging (`launcher.py`)

Redirect stderr to a log file instead of DEVNULL for diagnostics:

```python
log_path = session_dir / "gdb.log"
log_fd = open(str(log_path), "w")
process = subprocess.Popen(
    gdb_args,
    stdin=fifo_fd,
    stdout=subprocess.DEVNULL,
    stderr=log_fd,
    start_new_session=True,
)
os.close(fifo_fd)
log_fd.close()  # Parent process closes fd after child inherits it
```

Log file path: `~/.gdb-cli/sessions/<session_id>/gdb.log`

## Error Handling

| Scenario | Detection | Response |
|----------|-----------|----------|
| GDB crashes during load | `status` → socket connect fails, `gdb_pid` dead | Report session dead |
| Binary path invalid | GDB exits, socket connect fails | Report session dead |
| Core file corrupted | GDB may crash or hang | `status` shows loading; agent checks gdb.log |
| Socket creation fails | `_wait_for_socket` times out | Raise `GDBLauncherError` as before |

## Files Changed

| File | Changes |
|------|---------|
| `src/gdb_cli/launcher.py` | Reorder commands, add timeout param to `_start_gdb_process`, stderr to log file, close log_fd |
| `src/gdb_cli/gdb_server/gdb_rpc_server.py` | Add `_state`, `set_ready()`, loading guard in `_dispatch`, `_handle_loading_status`, inject server into `__main__` |
| `src/gdb_cli/cli.py` | Load returns "loading", status fallback with process liveness check |

## Usage Flow (AI Agent)

```
$ gdb-cli load -b ./observer -c ./core.xxx
{"session_id": "abc12345", "status": "loading", "gdb_pid": 12345}

# Agent does other work, then polls:
$ gdb-cli status -s abc12345
{"session_id": "abc12345", "state": "loading", "elapsed": 45.2}

$ gdb-cli status -s abc12345
{"session_id": "abc12345", "state": "ready"}

# Now agent can query:
$ gdb-cli bt -s abc12345
{...backtrace data...}
```

## What We Decided NOT To Do

- **`set auto-solib-add off`**: For large statically-linked binaries (~8GB), external .so are minimal (libc, libpthread). Benefit ~2-5s, not worth the added complexity of manual `sharedlibrary` loading.
- **`--wait` synchronous mode**: The only caller is AI agents, which naturally support async polling. YAGNI.
- **Merging python `-ex` commands**: Saves <5ms. Not worth the code change.
