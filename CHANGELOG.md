# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.1.0] - 2025-03-21

### Added
- Initial release
- Core dump analysis with symbol resident in memory
- Live attach debugging with non-stop mode support
- Structured JSON output for all commands
- Command whitelist security mechanism
- Heartbeat timeout auto-cleanup
- Session management with idempotency
- SSH remote debugging support
- Multi-language documentation (English, Chinese)

### Commands
- `load` - Load core dump files
- `attach` - Attach to running processes
- `threads` - List threads
- `bt` - Get backtrace
- `eval-cmd` - Evaluate C/C++ expressions
- `eval-element` - Access array/container elements
- `exec` - Execute raw GDB commands
- `thread-apply` - Batch thread operations
- `locals-cmd` - View local variables
- `status` - Check session status
- `sessions` - List active sessions
- `stop` - Stop a session
- `env-check` - Environment validation