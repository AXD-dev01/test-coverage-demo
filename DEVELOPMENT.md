# Development Guide

Complete guide for setting up and working with the test-coverage-demo project.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Initial Setup](#initial-setup)
- [Project Structure](#project-structure)
- [Development Workflow](#development-workflow)
- [Running Tests](#running-tests)
- [Code Quality Tools](#code-quality-tools)
- [Debugging](#debugging)
- [Common Tasks](#common-tasks)
- [Troubleshooting](#troubleshooting)

## Prerequisites

### Required Software

- **Python**: 3.10, 3.11, or 3.12
- **pip**: Latest version
- **Git**: For version control
- **Virtual environment**: venv (included with Python)

### Optional Tools

- **GitHub CLI** (`gh`): For PR management
- **IDE**: VS Code, PyCharm, or your preferred editor
- **Docker**: For containerized development (optional)

### Verify Installation

```bash
# Check Python version
python3 --version  # Should be 3.10+

# Check pip
pip3 --version

# Check git
git --version
```

## Initial Setup

### 1. Clone the Repository

```bash
# Clone via HTTPS
git clone https://github.com/AXD-dev01/test-coverage-demo.git
cd test-coverage-demo

# Or clone via SSH
git clone git@github.com:AXD-dev01/test-coverage-demo.git
cd test-coverage-demo
```

### 2. Create Virtual Environment

```bash
# Create virtual environment
python3 -m venv venv

# Activate virtual environment
# On macOS/Linux:
source venv/bin/activate

# On Windows:
venv\Scripts\activate

# You should see (venv) in your prompt
```

### 3. Install Dependencies

```bash
# Upgrade pip
pip install --upgrade pip

# Install test dependencies
pip install -r requirements-test.txt

# Verify installation
pip list
```

### 4. Verify Setup

```bash
# Run tests
pytest -v

# Check coverage
pytest --cov=src --cov-report=term-missing

# Should see 100% coverage
```

## Project Structure

```
test-coverage-demo/
├── .github/
│   └── workflows/
│       └── test.yml          # GitHub Actions CI/CD
├── src/
│   ├── __init__.py
│   └── calculator.py         # Main calculator module
├── tests/
│   ├── __init__.py
│   ├── test_example.py       # Test suite
│   ├── unit/                 # Unit tests (future)
│   └── integration/          # Integration tests (future)
├── venv/                     # Virtual environment (not in git)
├── htmlcov/                  # Coverage HTML reports (not in git)
├── .coveragerc               # Coverage configuration
├── .gitignore                # Git ignore patterns
├── codecov.yml               # Codecov configuration
├── pytest.ini                # Pytest configuration
├── requirements-test.txt     # Test dependencies
├── CONTRIBUTING.md           # Contribution guidelines
├── DEVELOPMENT.md            # This file
├── CODECOV.md                # Codecov documentation
├── TESTING.md                # Testing guide
├── CI_CD.md                  # CI/CD documentation
└── README.md                 # Project overview
```

### Key Files

**Source Code**:
- `src/calculator.py`: Main calculator implementation
- `src/__init__.py`: Package initialization

**Tests**:
- `tests/test_example.py`: Comprehensive test suite
- `tests/__init__.py`: Test package initialization

**Configuration**:
- `pytest.ini`: Pytest settings
- `.coveragerc`: Coverage.py settings
- `codecov.yml`: Codecov configuration
- `.github/workflows/test.yml`: CI/CD pipeline

## Development Workflow

### Daily Development Cycle

1. **Start work**:
   ```bash
   # Activate virtual environment
   source venv/bin/activate

   # Update from main
   git checkout main
   git pull origin main

   # Create feature branch
   git checkout -b feature/my-feature
   ```

2. **Make changes**:
   - Edit code in `src/`
   - Add/update tests in `tests/`
   - Run tests frequently

3. **Test changes**:
   ```bash
   # Quick test run
   pytest -v

   # With coverage
   pytest --cov=src

   # Specific test
   pytest tests/test_example.py::test_add -v
   ```

4. **Check code quality**:
   ```bash
   # Format code
   black src tests

   # Lint code
   flake8 src tests

   # Type check
   mypy src
   ```

5. **Commit changes**:
   ```bash
   git add .
   git commit -m "feat: add my feature"
   ```

6. **Push and create PR**:
   ```bash
   git push origin feature/my-feature
   gh pr create
   ```

### Branch Strategy

- **main**: Production-ready code
- **feature/***: New features
- **fix/***: Bug fixes
- **docs/***: Documentation updates
- **refactor/***: Code refactoring

## Running Tests

### Basic Test Commands

```bash
# Run all tests
pytest

# Verbose output
pytest -v

# Run specific test file
pytest tests/test_example.py

# Run specific test function
pytest tests/test_example.py::test_add

# Stop on first failure
pytest -x

# Show local variables on failure
pytest -l
```

### Coverage Commands

```bash
# Generate coverage report
pytest --cov=src

# Show missing lines
pytest --cov=src --cov-report=term-missing

# Generate HTML report
pytest --cov=src --cov-report=html

# Open HTML report
open htmlcov/index.html  # macOS
# or
xdg-open htmlcov/index.html  # Linux
```

### Advanced Testing

```bash
# Run tests in parallel
pytest -n auto

# Generate JUnit XML
pytest --junitxml=junit.xml

# Run with specific Python version
python3.10 -m pytest
python3.11 -m pytest
python3.12 -m pytest

# Measure test duration
pytest --durations=10
```

See [TESTING.md](TESTING.md) for comprehensive testing guide.

## Code Quality Tools

### Black (Code Formatting)

```bash
# Format all code
black src tests

# Check without modifying
black --check src tests

# Format specific file
black src/calculator.py

# Show diff
black --diff src tests
```

**Configuration**: Uses Black defaults (88 char line length)

### Flake8 (Linting)

```bash
# Lint all code
flake8 src tests

# Check specific file
flake8 src/calculator.py

# Show statistics
flake8 --statistics src tests

# Generate report
flake8 --output-file=flake8-report.txt src tests
```

**Configuration**: Defined in `.flake8` or `setup.cfg`

### mypy (Type Checking)

```bash
# Type check source code
mypy src

# Check specific file
mypy src/calculator.py

# Strict mode
mypy --strict src

# Generate report
mypy src --html-report mypy-report
```

**Configuration**: Defined in `mypy.ini` or `pyproject.toml`

### isort (Import Sorting)

```bash
# Sort imports
isort src tests

# Check only
isort --check-only src tests

# Show diff
isort --diff src tests
```

### Run All Quality Checks

```bash
# Combined command
black src tests && flake8 src tests && mypy src && pytest --cov=src
```

## Debugging

### Using pdb

```python
# Add breakpoint in code
import pdb; pdb.set_trace()

# Or use built-in (Python 3.7+)
breakpoint()
```

### Debugging Tests

```bash
# Run pytest with pdb
pytest --pdb

# Drop into pdb on first failure
pytest -x --pdb

# Drop into pdb on test start
pytest --trace
```

### VS Code Debugging

Create `.vscode/launch.json`:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Python: Current File",
      "type": "python",
      "request": "launch",
      "program": "${file}",
      "console": "integratedTerminal"
    },
    {
      "name": "Python: Pytest",
      "type": "python",
      "request": "launch",
      "module": "pytest",
      "args": ["-v"],
      "console": "integratedTerminal"
    }
  ]
}
```

## Common Tasks

### Adding a New Function

1. **Add function to `src/calculator.py`**:
   ```python
   def power(base, exponent):
       """Calculate base raised to exponent."""
       return base ** exponent
   ```

2. **Add tests to `tests/test_example.py`**:
   ```python
   def test_power():
       assert power(2, 3) == 8
       assert power(5, 2) == 25
   ```

3. **Run tests**:
   ```bash
   pytest -v
   ```

4. **Check coverage**:
   ```bash
   pytest --cov=src --cov-report=term-missing
   ```

### Updating Dependencies

```bash
# Update specific package
pip install --upgrade pytest

# Update all packages
pip list --outdated
pip install --upgrade <package>

# Update requirements file
pip freeze > requirements-test.txt
```

### Generating Documentation

```bash
# Install sphinx (if needed)
pip install sphinx

# Generate docs
sphinx-quickstart docs
sphinx-build -b html docs docs/_build
```

## Troubleshooting

### Virtual Environment Issues

**Problem**: `command not found: pytest`

**Solution**:
```bash
# Ensure venv is activated
source venv/bin/activate

# Reinstall dependencies
pip install -r requirements-test.txt
```

### Import Errors

**Problem**: `ModuleNotFoundError: No module named 'src'`

**Solution**:
```bash
# Install package in editable mode
pip install -e .

# Or set PYTHONPATH
export PYTHONPATH="${PYTHONPATH}:${PWD}"
```

### Coverage Not Working

**Problem**: Coverage shows 0%

**Solution**:
```bash
# Check coverage configuration
cat .coveragerc

# Use explicit source
pytest --cov=src --cov-config=.coveragerc
```

### Git Hooks Blocking Commits

**Problem**: Pre-commit hook fails

**Solution**:
```bash
# Fix the issue the hook is complaining about
# Or bypass (not recommended)
git commit --no-verify
```

### CI Failures

**Problem**: Tests pass locally but fail in CI

**Solution**:
```bash
# Check Python version matches CI
python --version

# Run full test suite locally
pytest --cov=src --cov-report=term-missing

# Check all quality tools
black --check src tests
flake8 src tests
mypy src
```

## Environment Variables

Optional environment variables:

```bash
# Codecov token (for local testing)
export CODECOV_TOKEN="your-token-here"

# Python path
export PYTHONPATH="${PWD}/src:${PYTHONPATH}"

# Test verbosity
export PYTEST_ADDOPTS="-v --tb=short"
```

## IDE Setup

### VS Code

Recommended extensions:
- Python (Microsoft)
- Pylance
- Python Test Explorer
- Coverage Gutters
- Black Formatter

Settings (`.vscode/settings.json`):
```json
{
  "python.linting.enabled": true,
  "python.linting.flake8Enabled": true,
  "python.formatting.provider": "black",
  "python.testing.pytestEnabled": true,
  "editor.formatOnSave": true
}
```

### PyCharm

1. Set Python interpreter to venv
2. Enable pytest as test runner
3. Configure Black as external tool
4. Enable type checking

## Additional Resources

- [CONTRIBUTING.md](CONTRIBUTING.md): Contribution guidelines
- [TESTING.md](TESTING.md): Testing guide
- [CODECOV.md](CODECOV.md): Codecov documentation
- [CI_CD.md](CI_CD.md): CI/CD pipeline details
- [Python Documentation](https://docs.python.org/3/)
- [Pytest Documentation](https://docs.pytest.org/)
- [Black Documentation](https://black.readthedocs.io/)

---

Happy coding! 🚀
