# GDB-CLI Skills

This directory contains Claude Code skills for intelligent debugging with gdb-cli.

## Available Skills

### gdb-cli

A debugging assistant that combines source code analysis with runtime state inspection.

**Features:**
- Core dump analysis with source code correlation
- Live process debugging with safety considerations
- Deadlock detection and analysis
- Memory corruption investigation
- Race condition diagnosis

**Install:**

```bash
bunx skills add https://github.com/Cerdore/gdb-cli --skill=gdb-cli
```

**Usage:**

```
/gdb-cli

# Or describe your debugging need:
I have a core dump at ./core.1234 and binary at ./myapp. Help me debug it.
```

## Prerequisites

- **gdb-cli**: `pip install git+https://github.com/Cerdore/gdb-cli.git`
- **GDB 9.0+** with Python support enabled
- **Linux** (recommended)

## Skill Development

Skills are evaluated using test cases in `evals/` directory. To run evaluations:

```bash
# See skill-creator documentation for evaluation workflow
```

## Adding New Skills

1. Create a new directory under `skills/`
2. Add a `SKILL.md` file with:
   - YAML frontmatter (name, description)
   - Markdown instructions for the skill
3. Add test cases in `skills/<name>/evals/evals.json`