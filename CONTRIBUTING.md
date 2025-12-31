# Contributing to Test Coverage Demo

Thank you for your interest in contributing! This document provides guidelines and instructions for contributing to this project.

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [Getting Started](#getting-started)
- [Development Workflow](#development-workflow)
- [Code Quality Standards](#code-quality-standards)
- [Testing Requirements](#testing-requirements)
- [Pull Request Process](#pull-request-process)
- [Commit Message Guidelines](#commit-message-guidelines)

## Code of Conduct

- Be respectful and inclusive
- Provide constructive feedback
- Focus on what is best for the community
- Show empathy towards other community members

## Getting Started

### Prerequisites

- Python 3.10, 3.11, or 3.12
- Git
- Virtual environment tool (venv)

### Setup Development Environment

1. **Fork and clone the repository**:
   ```bash
   git clone https://github.com/AXD-dev01/test-coverage-demo.git
   cd test-coverage-demo
   ```

2. **Create a virtual environment**:
   ```bash
   python3 -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   ```

3. **Install dependencies**:
   ```bash
   pip install -r requirements-test.txt
   ```

4. **Verify setup**:
   ```bash
   pytest -v
   ```

See [DEVELOPMENT.md](DEVELOPMENT.md) for detailed setup instructions.

## Development Workflow

1. **Create a feature branch**:
   ```bash
   git checkout -b feature/your-feature-name
   ```

2. **Make your changes**:
   - Write code following our style guidelines
   - Add tests for new functionality
   - Update documentation as needed

3. **Run tests locally**:
   ```bash
   # Run all tests
   pytest -v

   # Run with coverage
   pytest --cov=src --cov-report=html

   # Run specific test
   pytest tests/test_example.py::test_add -v
   ```

4. **Check code quality**:
   ```bash
   # Format code
   black src tests

   # Check linting
   flake8 src tests

   # Type checking
   mypy src
   ```

5. **Commit your changes**:
   ```bash
   git add .
   git commit -m "feat: add new feature"
   ```

6. **Push and create PR**:
   ```bash
   git push origin feature/your-feature-name
   ```

## Code Quality Standards

### Code Style

- **Formatting**: We use [Black](https://github.com/psf/black) for code formatting
  ```bash
  black src tests
  ```

- **Linting**: We use [Flake8](https://flake8.pycqa.org/) for linting
  ```bash
  flake8 src tests
  ```

- **Type Hints**: We use [mypy](http://mypy-lang.org/) for type checking
  ```bash
  mypy src
  ```

### Code Quality Checklist

Before submitting a PR, ensure:

- [ ] Code is formatted with Black
- [ ] No linting errors (flake8)
- [ ] Type hints are added and pass mypy
- [ ] All tests pass
- [ ] Code coverage is maintained or improved
- [ ] Documentation is updated
- [ ] Commit messages follow guidelines

## Testing Requirements

### Coverage Requirements

- **Minimum coverage**: 80% (enforced by CI)
- **Target coverage**: 100%
- **New code coverage**: 85% minimum

### Writing Tests

1. **Place tests in the `tests/` directory**
2. **Use descriptive test names**:
   ```python
   def test_add_positive_numbers():
       assert add(2, 3) == 5
   ```

3. **Follow AAA pattern**:
   ```python
   def test_divide_by_zero():
       # Arrange
       numerator = 10
       denominator = 0

       # Act & Assert
       with pytest.raises(ValueError):
           divide(numerator, denominator)
   ```

4. **Test edge cases**:
   - Boundary values
   - Error conditions
   - Edge cases specific to your function

See [TESTING.md](TESTING.md) for detailed testing guidelines.

## Pull Request Process

### Before Submitting

1. **Update your branch**:
   ```bash
   git checkout main
   git pull origin main
   git checkout your-feature-branch
   git rebase main
   ```

2. **Run full test suite**:
   ```bash
   pytest --cov=src --cov-report=term-missing
   ```

3. **Ensure CI will pass**:
   - All tests pass locally
   - Code is formatted (black)
   - No linting errors (flake8)
   - Type checks pass (mypy)
   - Coverage meets requirements

### PR Submission

1. **Create descriptive PR title**:
   - `feat: add new calculator function`
   - `fix: handle division by zero correctly`
   - `docs: update README with examples`

2. **Fill out PR template** (if provided)

3. **Link related issues**:
   ```
   Closes #123
   Related to #456
   ```

4. **Request review** from maintainers

### PR Review Process

- Maintainers will review within 2-3 business days
- Address review comments promptly
- Keep PR focused and reasonably sized
- CI must pass before merge

### After PR Approval

- Squash commits if requested
- Maintainer will merge using "Squash and merge"
- Delete your feature branch after merge

## Commit Message Guidelines

We follow [Conventional Commits](https://www.conventionalcommits.org/).

### Format

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Types

- **feat**: New feature
- **fix**: Bug fix
- **docs**: Documentation changes
- **style**: Code style changes (formatting, no logic change)
- **refactor**: Code refactoring
- **test**: Adding or updating tests
- **chore**: Maintenance tasks

### Examples

```
feat: add multiply function to calculator

Implement multiplication operation with support for
positive and negative numbers.

Closes #42
```

```
fix: handle division by zero

Add ValueError when attempting to divide by zero.
Update tests to verify error handling.
```

```
docs: update README with usage examples

Add code examples for all calculator functions.
```

### Commit Message Rules

- Use present tense ("add feature" not "added feature")
- Use imperative mood ("move cursor to..." not "moves cursor to...")
- First line should be 50 characters or less
- Reference issues and PRs in footer

## Issue Reporting

### Bug Reports

Include:
- Python version
- Operating system
- Steps to reproduce
- Expected vs actual behavior
- Error messages/stack traces
- Minimal reproducible example

### Feature Requests

Include:
- Use case description
- Proposed solution
- Alternative solutions considered
- Additional context

## Questions?

- Check existing [Issues](https://github.com/AXD-dev01/test-coverage-demo/issues)
- Review [Documentation](README.md)
- Ask in [Discussions](https://github.com/AXD-dev01/test-coverage-demo/discussions)

## License

By contributing, you agree that your contributions will be licensed under the MIT License.

---

Thank you for contributing! 🎉
