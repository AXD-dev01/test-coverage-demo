# Test Coverage Demo

[![codecov](https://codecov.io/gh/AXD-dev01/test-coverage-demo/branch/master/graph/badge.svg)](https://codecov.io/gh/AXD-dev01/test-coverage-demo)
[![Python CI with Coverage](https://github.com/AXD-dev01/test-coverage-demo/actions/workflows/test.yml/badge.svg)](https://github.com/AXD-dev01/test-coverage-demo/actions/workflows/test.yml)

Demo project showcasing test coverage with Codecov integration

## Features

- Python test coverage with pytest
- Codecov integration for coverage reporting
- GitHub Actions CI/CD pipeline
- Multiple Python version testing (3.10, 3.11, 3.12)
- 100% test coverage

## Setup

1. Clone the repository:
```bash
git clone https://github.com/AXD-dev01/test-coverage-demo.git
cd test-coverage-demo
```

2. Create and activate a virtual environment:
```bash
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

3. Install dependencies:
```bash
pip install -r requirements-test.txt
```

## Running Tests

Run tests with coverage:
```bash
pytest --cov=src --cov-report=html --cov-report=term-missing
```

View HTML coverage report:
```bash
open htmlcov/index.html  # On macOS
# Or navigate to htmlcov/index.html in your browser
```

## Project Structure

```
test-coverage-demo/
├── src/
│   ├── __init__.py
│   └── calculator.py       # Example calculator module
├── tests/
│   ├── __init__.py
│   ├── test_example.py     # Test suite with 100% coverage
│   ├── unit/
│   └── integration/
├── .github/
│   └── workflows/
│       └── test.yml        # GitHub Actions CI workflow
├── codecov.yml             # Codecov configuration
├── pytest.ini              # Pytest configuration
├── .coveragerc             # Coverage.py configuration
└── requirements-test.txt   # Test dependencies
```

## Coverage Reports

- **Codecov Dashboard**: https://app.codecov.io/github/AXD-dev01/test-coverage-demo
- **GitHub Actions**: https://github.com/AXD-dev01/test-coverage-demo/actions

## Example Calculator Module

The project includes a simple calculator module with comprehensive tests:

```python
from src.calculator import add, subtract, multiply, divide

# Basic operations
add(2, 3)        # Returns: 5
subtract(10, 4)  # Returns: 6
multiply(3, 7)   # Returns: 21
divide(20, 5)    # Returns: 4.0
```

## CI/CD Pipeline

Every push and pull request triggers:
1. Code linting with flake8
2. Code formatting check with black
3. Type checking with mypy
4. Test execution across Python 3.10, 3.11, and 3.12
5. Coverage report upload to Codecov
6. Test results and analytics upload to Codecov
7. Coverage artifacts storage

## License

MIT
