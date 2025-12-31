# Testing Guide

Comprehensive guide to testing practices, pytest usage, and achieving high code coverage in the test-coverage-demo project.

## Table of Contents

- [Testing Philosophy](#testing-philosophy)
- [Test Organization](#test-organization)
- [Getting Started with pytest](#getting-started-with-pytest)
- [Writing Effective Tests](#writing-effective-tests)
- [Test Coverage](#test-coverage)
- [Test Types](#test-types)
- [Advanced pytest Features](#advanced-pytest-features)
- [Testing Best Practices](#testing-best-practices)
- [Common Testing Patterns](#common-testing-patterns)
- [Mocking and Fixtures](#mocking-and-fixtures)
- [Testing Edge Cases](#testing-edge-cases)
- [Performance Testing](#performance-testing)
- [Continuous Testing](#continuous-testing)
- [Debugging Failed Tests](#debugging-failed-tests)
- [Troubleshooting](#troubleshooting)

## Testing Philosophy

### Why We Test

1. **Catch bugs early**: Find issues before they reach production
2. **Enable refactoring**: Safely change code with confidence
3. **Document behavior**: Tests serve as executable documentation
4. **Improve design**: Writing tests often reveals design issues
5. **Reduce debugging time**: Failing tests pinpoint problems quickly

### Our Testing Standards

- **100% coverage target**: We aim for complete test coverage
- **80% minimum coverage**: Required by our quality standards
- **85% for new code**: Higher bar for patches and new features
- **Test all edge cases**: Including error conditions and boundary values
- **Fast tests**: Keep tests quick to encourage frequent running

### Test-Driven Development (TDD)

We encourage (but don't require) TDD:

1. **Write test first**: Define expected behavior
2. **Run test (it fails)**: Verify test actually tests something
3. **Write minimal code**: Make test pass
4. **Refactor**: Improve code while keeping tests green
5. **Repeat**: Move to next feature

## Test Organization

### Directory Structure

```
test-coverage-demo/
├── src/
│   ├── __init__.py
│   └── calculator.py         # Source code
├── tests/
│   ├── __init__.py
│   ├── test_example.py       # Current test file
│   ├── unit/                 # Unit tests (future)
│   │   ├── __init__.py
│   │   └── test_calculator.py
│   ├── integration/          # Integration tests (future)
│   │   ├── __init__.py
│   │   └── test_workflow.py
│   └── fixtures/             # Test data (future)
│       └── sample_data.py
```

### Naming Conventions

**Test Files**:
- Prefix with `test_`: `test_calculator.py`
- Or suffix with `_test`: `calculator_test.py`
- Match source file names when possible

**Test Functions**:
- Prefix with `test_`: `test_add_positive_numbers()`
- Be descriptive: Name should explain what's being tested
- Use underscores for readability

**Test Classes** (optional):
- Prefix with `Test`: `class TestCalculator:`
- Group related tests together
- Methods must start with `test_`

### File Organization

```python
# tests/test_calculator.py

# 1. Imports
import pytest
from src.calculator import add, subtract

# 2. Fixtures (if needed)
@pytest.fixture
def sample_data():
    return {"a": 5, "b": 3}

# 3. Test classes (optional grouping)
class TestArithmetic:
    def test_addition(self):
        assert add(2, 3) == 5

    def test_subtraction(self):
        assert subtract(5, 3) == 2

# 4. Standalone test functions
def test_edge_case():
    assert add(0, 0) == 0
```

## Getting Started with pytest

### Installation

```bash
# Activate virtual environment
source venv/bin/activate

# Install pytest and coverage
pip install pytest pytest-cov
```

### Basic Usage

```bash
# Run all tests
pytest

# Verbose output
pytest -v

# Run specific file
pytest tests/test_calculator.py

# Run specific test
pytest tests/test_calculator.py::test_add

# Run tests matching pattern
pytest -k "add"

# Stop on first failure
pytest -x

# Show local variables on failure
pytest -l

# Run last failed tests
pytest --lf

# Run failed tests first, then others
pytest --ff
```

### pytest Configuration

**File**: `pytest.ini` (in project root)

```ini
[pytest]
# Pytest configuration
testpaths = tests
python_files = test_*.py *_test.py
python_classes = Test*
python_functions = test_*

# Output options
addopts =
    -v
    --strict-markers
    --tb=short
    --cov=src
    --cov-report=term-missing
    --cov-report=html
    --cov-branch

# Markers
markers =
    slow: marks tests as slow (deselect with '-m "not slow"')
    integration: marks tests as integration tests
    unit: marks tests as unit tests
```

## Writing Effective Tests

### The AAA Pattern

**Arrange-Act-Assert**: Standard test structure

```python
def test_divide():
    # Arrange: Set up test data
    numerator = 10
    denominator = 2
    expected = 5

    # Act: Execute the function
    result = divide(numerator, denominator)

    # Assert: Verify the result
    assert result == expected
```

### Simple Assertions

```python
# Equality
assert add(2, 3) == 5

# Inequality
assert subtract(5, 3) != 3

# Boolean
assert is_even(4) is True
assert is_positive(-5) is False

# Membership
assert 2 in [1, 2, 3]
assert "hello" not in "world"

# Comparisons
assert 5 > 3
assert 10 >= 10
assert 2 < 5
assert 3 <= 3

# Identity
assert my_var is None
assert my_var is not None

# Type checking
assert isinstance(result, int)
assert isinstance(name, str)
```

### Testing Exceptions

```python
def test_divide_by_zero():
    """Test that dividing by zero raises ValueError"""
    with pytest.raises(ValueError):
        divide(10, 0)

def test_divide_by_zero_with_message():
    """Test exception message"""
    with pytest.raises(ValueError, match="Cannot divide by zero"):
        divide(10, 0)

def test_exception_details():
    """Capture and inspect exception"""
    with pytest.raises(ValueError) as exc_info:
        divide(10, 0)

    assert "zero" in str(exc_info.value)
    assert exc_info.type is ValueError
```

### Testing Multiple Cases

```python
# Multiple assertions (simple)
def test_add_multiple_cases():
    assert add(2, 3) == 5
    assert add(-1, 1) == 0
    assert add(0, 0) == 0
    assert add(-5, -3) == -8

# Parametrized tests (better)
@pytest.mark.parametrize("a, b, expected", [
    (2, 3, 5),
    (-1, 1, 0),
    (0, 0, 0),
    (-5, -3, -8),
])
def test_add_parametrized(a, b, expected):
    assert add(a, b) == expected
```

### Descriptive Test Names

```python
# Bad: Not descriptive
def test_add():
    assert add(2, 3) == 5

# Good: Describes what's being tested
def test_add_positive_numbers():
    assert add(2, 3) == 5

def test_add_negative_numbers():
    assert add(-2, -3) == -5

def test_add_mixed_signs():
    assert add(5, -3) == 2

def test_add_with_zero():
    assert add(0, 5) == 5
```

### Docstrings

```python
def test_divide_by_zero():
    """
    Test that divide() raises ValueError when denominator is zero.

    The function should raise a ValueError with a descriptive message
    when attempting to divide by zero, as this is mathematically undefined.
    """
    with pytest.raises(ValueError, match="Cannot divide by zero"):
        divide(10, 0)
```

## Test Coverage

### Generating Coverage Reports

```bash
# Terminal report
pytest --cov=src

# Terminal with missing lines
pytest --cov=src --cov-report=term-missing

# HTML report
pytest --cov=src --cov-report=html

# XML report (for Codecov)
pytest --cov=src --cov-report=xml

# Multiple reports
pytest --cov=src --cov-report=term-missing --cov-report=html --cov-report=xml
```

### Understanding Coverage Output

```
---------- coverage: platform darwin, python 3.11.5 -----------
Name                    Stmts   Miss  Cover   Missing
-----------------------------------------------------
src/__init__.py             0      0   100%
src/calculator.py          15      0   100%
-----------------------------------------------------
TOTAL                      15      0   100%
```

**Columns**:
- **Name**: File path
- **Stmts**: Total statements (lines of code)
- **Miss**: Statements not covered by tests
- **Cover**: Coverage percentage
- **Missing**: Line numbers not covered

### Branch Coverage

```bash
# Enable branch coverage
pytest --cov=src --cov-branch --cov-report=term-missing
```

**Branch coverage** tracks if all conditional paths are tested:

```python
def is_positive(n):
    if n > 0:        # Branch: True path
        return True
    return False      # Branch: False path
```

**Both paths must be tested** for 100% branch coverage:

```python
def test_is_positive_true():
    assert is_positive(5) is True  # Tests True branch

def test_is_positive_false():
    assert is_positive(-3) is False  # Tests False branch
```

### Coverage Configuration

**File**: `.coveragerc`

```ini
[run]
source = src
omit =
    */tests/*
    */venv/*
    */__pycache__/*

[report]
precision = 2
show_missing = True
skip_covered = False

[html]
directory = htmlcov
```

### Viewing HTML Coverage

```bash
# Generate HTML report
pytest --cov=src --cov-report=html

# Open in browser
open htmlcov/index.html  # macOS
xdg-open htmlcov/index.html  # Linux
start htmlcov/index.html  # Windows
```

**HTML Report Features**:
- Click files to see line-by-line coverage
- Green lines: Covered
- Red lines: Not covered
- Yellow lines: Partially covered (some branches missed)

### Coverage Goals

- **100%**: Our target (currently achieved!)
- **80%**: Minimum required by Codecov
- **85%**: Required for new code (patches)
- **90%**: Required for critical paths (core, api, security)

## Test Types

### Unit Tests

**Purpose**: Test individual functions/methods in isolation

**Characteristics**:
- Fast execution
- No external dependencies
- Test one thing at a time
- Easy to debug

**Example**:
```python
def test_add_unit():
    """Unit test for add() function"""
    result = add(2, 3)
    assert result == 5
```

**Location**: `tests/unit/`

### Integration Tests

**Purpose**: Test how components work together

**Characteristics**:
- Test interactions between components
- May use real dependencies
- Slower than unit tests
- Test realistic scenarios

**Example**:
```python
def test_calculator_workflow():
    """Integration test for calculator workflow"""
    # Test realistic usage
    result = add(10, 5)
    result = multiply(result, 2)
    result = divide(result, 3)
    assert result == 10
```

**Location**: `tests/integration/`

### Functional Tests

**Purpose**: Test complete features from user perspective

**Example**:
```python
def test_calculator_basic_operations():
    """Test all basic calculator operations work together"""
    # Addition
    assert add(2, 3) == 5

    # Subtraction
    assert subtract(5, 3) == 2

    # Multiplication
    assert multiply(3, 4) == 12

    # Division
    assert divide(10, 2) == 5
```

### Regression Tests

**Purpose**: Ensure bugs don't reappear

**Pattern**: When you fix a bug, write a test that would have caught it

```python
def test_divide_by_zero_regression():
    """
    Regression test for bug #42: divide() crashed on zero instead of raising error.

    Bug: divide(10, 0) caused program crash
    Fix: Now raises ValueError with descriptive message
    """
    with pytest.raises(ValueError, match="Cannot divide by zero"):
        divide(10, 0)
```

## Advanced pytest Features

### Fixtures

**Purpose**: Reusable test setup and teardown

```python
import pytest

@pytest.fixture
def sample_numbers():
    """Provide sample numbers for testing"""
    return {"a": 10, "b": 5, "c": 0}

def test_with_fixture(sample_numbers):
    """Test using fixture"""
    assert add(sample_numbers["a"], sample_numbers["b"]) == 15
```

### Fixture Scopes

```python
@pytest.fixture(scope="function")  # Default: New instance per test
def function_fixture():
    return setup_data()

@pytest.fixture(scope="class")  # Shared across test class
def class_fixture():
    return setup_data()

@pytest.fixture(scope="module")  # Shared across test file
def module_fixture():
    return setup_data()

@pytest.fixture(scope="session")  # Shared across entire test session
def session_fixture():
    return setup_data()
```

### Parametrized Tests

```python
@pytest.mark.parametrize("a, b, expected", [
    (2, 3, 5),
    (0, 0, 0),
    (-1, 1, 0),
    (10, -5, 5),
])
def test_add_parametrized(a, b, expected):
    assert add(a, b) == expected
```

**With IDs**:
```python
@pytest.mark.parametrize("a, b, expected", [
    pytest.param(2, 3, 5, id="positive"),
    pytest.param(0, 0, 0, id="zeros"),
    pytest.param(-1, 1, 0, id="negative"),
    pytest.param(10, -5, 5, id="mixed"),
])
def test_add_parametrized(a, b, expected):
    assert add(a, b) == expected
```

### Test Markers

```python
# Mark slow tests
@pytest.mark.slow
def test_complex_calculation():
    # Long-running test
    pass

# Mark integration tests
@pytest.mark.integration
def test_database_integration():
    pass

# Skip tests
@pytest.mark.skip(reason="Not implemented yet")
def test_future_feature():
    pass

# Skip conditionally
@pytest.mark.skipif(sys.version_info < (3, 10), reason="Requires Python 3.10+")
def test_new_python_feature():
    pass

# Expected failure
@pytest.mark.xfail(reason="Known bug #123")
def test_known_issue():
    pass
```

**Run specific markers**:
```bash
# Run only unit tests
pytest -m unit

# Run everything except slow tests
pytest -m "not slow"

# Run unit or integration tests
pytest -m "unit or integration"
```

### Fixtures with Cleanup

```python
@pytest.fixture
def temp_file():
    """Create temporary file and clean up after test"""
    # Setup
    file_path = "temp_test_file.txt"
    with open(file_path, "w") as f:
        f.write("test data")

    # Provide to test
    yield file_path

    # Cleanup (runs after test)
    if os.path.exists(file_path):
        os.remove(file_path)

def test_with_temp_file(temp_file):
    """Test that uses temporary file"""
    assert os.path.exists(temp_file)
    # temp_file will be deleted after this test
```

### Parametrized Fixtures

```python
@pytest.fixture(params=[2, 4, 6, 8])
def even_number(request):
    return request.param

def test_is_even_parametrized(even_number):
    """Test runs 4 times with different even numbers"""
    assert is_even(even_number) is True
```

## Testing Best Practices

### 1. Test One Thing at a Time

```python
# Bad: Tests multiple things
def test_calculator():
    assert add(2, 3) == 5
    assert subtract(5, 3) == 2
    assert multiply(3, 4) == 12
    assert divide(10, 2) == 5

# Good: Separate tests
def test_add():
    assert add(2, 3) == 5

def test_subtract():
    assert subtract(5, 3) == 2

def test_multiply():
    assert multiply(3, 4) == 12

def test_divide():
    assert divide(10, 2) == 5
```

### 2. Make Tests Independent

```python
# Bad: Tests depend on each other
result = None

def test_step_1():
    global result
    result = add(2, 3)
    assert result == 5

def test_step_2():
    global result
    result = multiply(result, 2)  # Depends on test_step_1
    assert result == 10

# Good: Each test is independent
def test_add():
    result = add(2, 3)
    assert result == 5

def test_multiply():
    result = multiply(5, 2)
    assert result == 10
```

### 3. Use Descriptive Names

```python
# Bad
def test_1():
    assert add(2, 3) == 5

# Good
def test_add_positive_integers():
    assert add(2, 3) == 5
```

### 4. Test Edge Cases

```python
def test_add_edge_cases():
    # Zero
    assert add(0, 5) == 5
    assert add(5, 0) == 5

    # Negative
    assert add(-5, 3) == -2

    # Large numbers
    assert add(999999, 1) == 1000000

    # Floats
    assert add(1.5, 2.5) == 4.0
```

### 5. Keep Tests Fast

```python
# Bad: Slow test
def test_slow():
    time.sleep(5)  # Avoid delays
    assert True

# Good: Fast test
def test_fast():
    assert add(2, 3) == 5  # Instant
```

### 6. Avoid Test Logic

```python
# Bad: Logic in test
def test_with_logic():
    result = add(2, 3)
    if result > 4:
        assert result == 5
    else:
        assert result == 4

# Good: Simple assertion
def test_simple():
    result = add(2, 3)
    assert result == 5
```

### 7. Use Fixtures for Setup

```python
# Bad: Repeated setup
def test_1():
    data = {"a": 5, "b": 3}
    assert add(data["a"], data["b"]) == 8

def test_2():
    data = {"a": 5, "b": 3}
    assert subtract(data["a"], data["b"]) == 2

# Good: Fixture
@pytest.fixture
def data():
    return {"a": 5, "b": 3}

def test_1(data):
    assert add(data["a"], data["b"]) == 8

def test_2(data):
    assert subtract(data["a"], data["b"]) == 2
```

## Common Testing Patterns

### Testing Error Handling

```python
def test_error_handling():
    """Test that function handles errors correctly"""
    with pytest.raises(ValueError):
        divide(10, 0)

    with pytest.raises(TypeError):
        add("string", 5)
```

### Testing Return Values

```python
def test_return_values():
    """Test various return value types"""
    # Integer
    assert isinstance(add(2, 3), int)

    # Float
    assert isinstance(divide(5, 2), float)

    # Boolean
    assert isinstance(is_even(4), bool)
```

### Testing State Changes

```python
def test_state_change():
    """Test that state changes correctly"""
    calculator = Calculator()
    assert calculator.memory == 0

    calculator.store(42)
    assert calculator.memory == 42

    calculator.clear()
    assert calculator.memory == 0
```

### Testing Side Effects

```python
def test_file_creation():
    """Test that function creates file"""
    file_path = "output.txt"

    # Ensure file doesn't exist
    if os.path.exists(file_path):
        os.remove(file_path)

    # Call function
    write_output(file_path, "data")

    # Verify file created
    assert os.path.exists(file_path)

    # Cleanup
    os.remove(file_path)
```

## Mocking and Fixtures

### When to Mock

- External APIs
- Database connections
- File system operations
- Time-dependent code
- Random number generation
- Expensive operations

### Using unittest.mock

```python
from unittest.mock import Mock, patch

def test_with_mock():
    """Test using mock objects"""
    # Create mock
    mock_api = Mock()
    mock_api.get_data.return_value = {"result": 42}

    # Use mock
    result = process_api_data(mock_api)

    # Verify
    assert result == 42
    mock_api.get_data.assert_called_once()
```

### Patching

```python
@patch('module.external_api_call')
def test_with_patch(mock_api):
    """Test using patch decorator"""
    mock_api.return_value = {"data": "value"}

    result = function_that_calls_api()

    assert result == "processed: value"
    mock_api.assert_called_once()
```

### pytest-mock

```bash
pip install pytest-mock
```

```python
def test_with_mocker(mocker):
    """Test using pytest-mock"""
    mock = mocker.patch('module.function')
    mock.return_value = 42

    result = call_function()

    assert result == 42
    mock.assert_called_once()
```

## Testing Edge Cases

### Boundary Values

```python
def test_boundary_values():
    """Test boundary conditions"""
    # Zero
    assert is_positive(0) is False

    # Just above zero
    assert is_positive(0.0001) is True

    # Just below zero
    assert is_positive(-0.0001) is False

    # Large positive
    assert is_positive(999999999) is True

    # Large negative
    assert is_positive(-999999999) is False
```

### Empty Inputs

```python
def test_empty_inputs():
    """Test with empty inputs"""
    assert sum_list([]) == 0
    assert max_value([]) is None
    assert process_string("") == ""
```

### Null/None Values

```python
def test_none_values():
    """Test handling of None"""
    with pytest.raises(TypeError):
        add(None, 5)

    assert safe_add(None, 5, default=0) == 5
```

### Type Mismatches

```python
def test_type_errors():
    """Test type validation"""
    with pytest.raises(TypeError):
        add("string", 5)

    with pytest.raises(TypeError):
        add([1, 2], 3)
```

## Performance Testing

### Using pytest-benchmark

```bash
pip install pytest-benchmark
```

```python
def test_add_performance(benchmark):
    """Benchmark add() function"""
    result = benchmark(add, 2, 3)
    assert result == 5
```

### Timing Tests

```python
import time

def test_fast_execution():
    """Test executes quickly"""
    start = time.time()

    result = add(2, 3)

    duration = time.time() - start
    assert duration < 0.1  # Should complete in < 100ms
    assert result == 5
```

### Using pytest-timeout

```bash
pip install pytest-timeout
```

```python
@pytest.mark.timeout(5)
def test_with_timeout():
    """Test must complete within 5 seconds"""
    result = expensive_operation()
    assert result is not None
```

## Continuous Testing

### Watch Mode (pytest-watch)

```bash
# Install
pip install pytest-watch

# Watch for changes and auto-run tests
ptw

# Watch with coverage
ptw -- --cov=src
```

### IDE Integration

**VS Code**:
1. Install Python extension
2. Open Command Palette (Cmd+Shift+P)
3. Select "Python: Configure Tests"
4. Choose pytest
5. Tests appear in Test Explorer

**PyCharm**:
1. Right-click test file
2. Select "Run pytest in test_example.py"
3. Or use Run menu > Run...

### Pre-commit Hooks

```bash
# Install pre-commit
pip install pre-commit

# Create .pre-commit-config.yaml
cat > .pre-commit-config.yaml << EOF
repos:
  - repo: local
    hooks:
      - id: pytest
        name: pytest
        entry: pytest
        language: system
        pass_filenames: false
        always_run: true
EOF

# Install hooks
pre-commit install
```

Now tests run automatically on `git commit`.

## Debugging Failed Tests

### Verbose Output

```bash
# Show detailed output
pytest -v

# Show local variables on failure
pytest -l

# Show full diff on assertion errors
pytest -vv
```

### Stop on First Failure

```bash
# Stop immediately on first failure
pytest -x

# Stop after N failures
pytest --maxfail=3
```

### Run Specific Tests

```bash
# Run one test
pytest tests/test_example.py::test_add

# Run tests matching pattern
pytest -k "add"

# Run tests by marker
pytest -m unit
```

### Interactive Debugging

```bash
# Drop into pdb on failure
pytest --pdb

# Drop into pdb on first failure
pytest -x --pdb

# Start pdb at beginning of test
pytest --trace
```

### Capture Output

```bash
# Show print statements
pytest -s

# Show captured output on failure
pytest --capture=no
```

### Reruns

```bash
# Rerun failed tests
pytest --lf

# Run failed first, then others
pytest --ff
```

## Troubleshooting

### Tests Not Discovered

**Problem**: pytest doesn't find tests

**Solutions**:
1. Check file naming: `test_*.py` or `*_test.py`
2. Check function naming: `test_*()`
3. Check pytest.ini `testpaths` setting
4. Run with discovery: `pytest --collect-only`

### Import Errors

**Problem**: `ModuleNotFoundError: No module named 'src'`

**Solutions**:
```bash
# Install in editable mode
pip install -e .

# Or set PYTHONPATH
export PYTHONPATH="${PWD}:${PYTHONPATH}"

# Or run from project root
cd /path/to/test-coverage-demo
pytest
```

### Coverage Shows 0%

**Problem**: Coverage report shows 0% despite tests passing

**Solutions**:
1. Check source path: `pytest --cov=src`
2. Verify .coveragerc configuration
3. Check that source files are in correct location
4. Use absolute paths in coverage config

### Fixture Not Found

**Problem**: `fixture 'my_fixture' not found`

**Solutions**:
1. Check fixture is defined in same file or conftest.py
2. Check fixture name spelling
3. Ensure conftest.py is in tests/ directory
4. Check fixture scope is appropriate

### Tests Passing Locally, Failing in CI

**Problem**: Tests work locally but fail in GitHub Actions

**Solutions**:
1. Check Python version matches
2. Check dependencies are installed
3. Check environment variables
4. Check file paths (use cross-platform paths)
5. Review CI logs carefully

### Slow Tests

**Problem**: Tests take too long

**Solutions**:
1. Run tests in parallel: `pytest -n auto`
2. Use markers to skip slow tests: `pytest -m "not slow"`
3. Profile tests: `pytest --durations=10`
4. Mock expensive operations
5. Reduce fixture scope

## Additional Resources

### Documentation

- **pytest**: https://docs.pytest.org/
- **Coverage.py**: https://coverage.readthedocs.io/
- **pytest-cov**: https://pytest-cov.readthedocs.io/
- **unittest.mock**: https://docs.python.org/3/library/unittest.mock.html

### Related Project Docs

- [CODECOV.md](CODECOV.md): Codecov integration guide
- [CI_CD.md](CI_CD.md): CI/CD pipeline documentation
- [DEVELOPMENT.md](DEVELOPMENT.md): Development setup
- [CONTRIBUTING.md](CONTRIBUTING.md): Contribution guidelines

### Useful Plugins

```bash
# Parallel execution
pip install pytest-xdist

# Benchmarking
pip install pytest-benchmark

# Coverage
pip install pytest-cov

# Mocking
pip install pytest-mock

# Timeout
pip install pytest-timeout

# Watch mode
pip install pytest-watch

# Randomly order tests
pip install pytest-randomly
```

---

**Last Updated**: December 31, 2025
**Current Coverage**: 100%
**Tests Passing**: 7/7
