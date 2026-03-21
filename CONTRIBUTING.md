# Contributing to GDB CLI

Thank you for your interest in contributing to GDB CLI for AI!

## Development Setup

```bash
# Clone the repository
git clone https://github.com/Cerdore/gdb-cli.git
cd gdb-cli

# Create virtual environment
python -m venv venv
source venv/bin/activate

# Install in development mode
pip install -e ".[dev]"
```

## Running Tests

```bash
# Run all tests
pytest tests/ -v

# Run with coverage
pytest tests/ --cov=src/gdb_cli --cov-report=html
```

## Code Style

We use [Ruff](https://docs.astral.sh/ruff/) for linting and formatting.

```bash
# Check for issues
ruff check src/ tests/

# Format code
ruff format src/ tests/
```

## Submitting Changes

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Make your changes
4. Run tests to ensure everything works
5. Commit your changes (`git commit -m 'Add amazing feature'`)
6. Push to your fork (`git push origin feature/amazing-feature`)
7. Open a Pull Request

## Pull Request Guidelines

- Write clear commit messages
- Add tests for new functionality
- Update documentation if needed
- Keep changes focused and minimal

## Reporting Issues

- Use the [GitHub issue tracker](https://github.com/Cerdore/gdb-cli/issues)
- Include Python version, GDB version, and OS
- Provide steps to reproduce the problem
- Include relevant logs or error messages

## Questions?

Feel free to open an issue for questions or discussions.