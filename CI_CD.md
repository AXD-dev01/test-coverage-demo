# CI/CD Pipeline Documentation

Complete guide to the Continuous Integration and Continuous Deployment (CI/CD) pipeline for the test-coverage-demo project.

## Table of Contents

- [Overview](#overview)
- [Pipeline Architecture](#pipeline-architecture)
- [Workflow Configuration](#workflow-configuration)
- [Pipeline Stages](#pipeline-stages)
- [Matrix Testing](#matrix-testing)
- [Code Quality Checks](#code-quality-checks)
- [Coverage and Test Reporting](#coverage-and-test-reporting)
- [Secrets Management](#secrets-management)
- [Artifacts](#artifacts)
- [Status Checks](#status-checks)
- [Troubleshooting CI Failures](#troubleshooting-ci-failures)
- [Best Practices](#best-practices)
- [Advanced Features](#advanced-features)
- [Monitoring and Metrics](#monitoring-and-metrics)

## Overview

### What is CI/CD?

**Continuous Integration (CI)**:
- Automatically test code changes
- Catch bugs early in development
- Ensure code quality standards
- Provide fast feedback to developers

**Continuous Deployment (CD)**:
- Automate deployment process
- Deploy to production safely
- Reduce manual errors
- Enable rapid releases

### Our CI/CD Stack

- **Platform**: GitHub Actions
- **Testing**: pytest + coverage.py
- **Linting**: flake8
- **Formatting**: black
- **Type Checking**: mypy
- **Coverage Reporting**: Codecov
- **Test Analytics**: Codecov Test Analytics
- **Python Versions**: 3.10, 3.11, 3.12

### Benefits

1. **Automated Testing**: Every commit is tested
2. **Multi-Version Support**: Test across Python versions
3. **Quality Enforcement**: Automated linting and formatting checks
4. **Coverage Tracking**: Automatic coverage reporting to Codecov
5. **Fast Feedback**: See results in minutes
6. **PR Protection**: Block merges that break tests or reduce coverage

## Pipeline Architecture

### Workflow File

**Location**: `.github/workflows/test.yml`

### Trigger Events

```yaml
on:
  push:
    branches: [ main, master, develop ]
  pull_request:
    branches: [ main, master, develop ]
```

**When Pipeline Runs**:
- **Push to main/master/develop**: Full pipeline executes
- **Pull Request to main/master/develop**: Full pipeline executes
- **All commits**: Get tested automatically

### Execution Environment

```yaml
runs-on: ubuntu-latest
```

**OS**: Ubuntu Linux (latest LTS)
**Virtual Machine**: GitHub-hosted runner
**Resources**: 2-core CPU, 7 GB RAM, 14 GB SSD

### Matrix Strategy

```yaml
strategy:
  matrix:
    python-version: ['3.10', '3.11', '3.12']
```

**Result**: Pipeline runs **3 times in parallel** (once per Python version)

## Workflow Configuration

### Complete Workflow

**File**: `.github/workflows/test.yml`

```yaml
name: Python CI with Coverage

on:
  push:
    branches: [ main, master, develop ]
  pull_request:
    branches: [ main, master, develop ]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ['3.10', '3.11', '3.12']

    steps:
    - name: Checkout code
      uses: actions/checkout@v4
      with:
        fetch-depth: 0

    - name: Set up Python ${{ matrix.python-version }}
      uses: actions/setup-python@v5
      with:
        python-version: ${{ matrix.python-version }}
        cache: 'pip'

    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt
        pip install -r requirements-test.txt

    - name: Lint with flake8
      run: |
        flake8 src --count --select=E9,F63,F7,F82 --show-source --statistics
        flake8 src --count --exit-zero --max-complexity=10 --max-line-length=127 --statistics

    - name: Format check with black
      run: |
        black --check src tests

    - name: Type check with mypy
      run: |
        mypy src
      continue-on-error: true

    - name: Run tests with pytest
      run: |
        pytest --cov=src --cov-report=xml --cov-report=term-missing --cov-branch --junitxml=junit.xml -o junit_family=legacy

    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v5
      with:
        token: ${{ secrets.CODECOV_TOKEN }}
        files: ./coverage.xml
        flags: python-${{ matrix.python-version }}
        name: codecov-python-${{ matrix.python-version }}
        fail_ci_if_error: false
        verbose: true

    - name: Upload test results to Codecov
      if: ${{ !cancelled() }}
      uses: codecov/codecov-action@v5
      with:
        token: ${{ secrets.CODECOV_TOKEN }}
        files: ./junit.xml
        flags: python-${{ matrix.python-version }}
        name: test-results-python-${{ matrix.python-version }}
        report_type: test_results
        verbose: true

    - name: Upload coverage reports as artifact
      uses: actions/upload-artifact@v4
      with:
        name: coverage-report-${{ matrix.python-version }}
        path: |
          coverage.xml
          htmlcov/
        retention-days: 30
```

## Pipeline Stages

### Stage 1: Checkout Code

```yaml
- name: Checkout code
  uses: actions/checkout@v4
  with:
    fetch-depth: 0
```

**Purpose**: Clone repository to runner

**Key Settings**:
- `fetch-depth: 0`: Fetch full git history (needed for coverage comparisons)
- Uses latest checkout action (v4)

**Why Full History?**
- Codecov needs to compare coverage with base branch
- Better diff analysis for PRs
- Access to commit history for annotations

### Stage 2: Set Up Python

```yaml
- name: Set up Python ${{ matrix.python-version }}
  uses: actions/setup-python@v5
  with:
    python-version: ${{ matrix.python-version }}
    cache: 'pip'
```

**Purpose**: Install specified Python version

**Key Features**:
- `${{ matrix.python-version }}`: Uses matrix value (3.10, 3.11, or 3.12)
- `cache: 'pip'`: Caches pip dependencies for faster runs
- Uses setup-python v5 (latest)

**Cache Benefits**:
- Faster dependency installation
- Reduced network usage
- More reliable builds

### Stage 3: Install Dependencies

```yaml
- name: Install dependencies
  run: |
    python -m pip install --upgrade pip
    pip install -r requirements.txt
    pip install -r requirements-test.txt
```

**Purpose**: Install project and test dependencies

**Steps**:
1. Upgrade pip to latest version
2. Install project dependencies (requirements.txt)
3. Install test dependencies (requirements-test.txt)

**Dependencies Installed**:
- pytest, pytest-cov
- flake8, black, mypy
- coverage[toml]
- All other test tools

### Stage 4: Lint with flake8

```yaml
- name: Lint with flake8
  run: |
    flake8 src --count --select=E9,F63,F7,F82 --show-source --statistics
    flake8 src --count --exit-zero --max-complexity=10 --max-line-length=127 --statistics
```

**Purpose**: Check code for style issues and errors

**Two Checks**:

1. **Critical Errors** (fails pipeline):
   - E9: Syntax errors
   - F63: Invalid print statement
   - F7: Syntax errors in docstrings
   - F82: Undefined names

2. **Style Warnings** (doesn't fail):
   - Max complexity: 10
   - Max line length: 127
   - exit-zero: Never fails

**Output**:
- Count of issues found
- Source code for each issue
- Statistics summary

### Stage 5: Format Check with black

```yaml
- name: Format check with black
  run: |
    black --check src tests
```

**Purpose**: Ensure code is formatted correctly

**Behavior**:
- `--check`: Check only, don't modify files
- Fails if any files would be reformatted
- Checks both `src/` and `tests/` directories

**To Fix Locally**:
```bash
black src tests
```

### Stage 6: Type Check with mypy

```yaml
- name: Type check with mypy
  run: |
    mypy src
  continue-on-error: true
```

**Purpose**: Static type checking

**Key Settings**:
- `continue-on-error: true`: Warning only, doesn't fail pipeline
- Only checks `src/` (not tests)

**Why Continue on Error?**
- Type hints are optional in Python
- Allows gradual adoption
- Provides warnings without blocking

### Stage 7: Run Tests with pytest

```yaml
- name: Run tests with pytest
  run: |
    pytest --cov=src --cov-report=xml --cov-report=term-missing --cov-branch --junitxml=junit.xml -o junit_family=legacy
```

**Purpose**: Execute test suite with coverage

**Flags Explained**:
- `--cov=src`: Measure coverage for src/ directory
- `--cov-report=xml`: Generate XML report for Codecov
- `--cov-report=term-missing`: Show missing lines in terminal
- `--cov-branch`: Enable branch coverage
- `--junitxml=junit.xml`: Generate JUnit XML for test results
- `-o junit_family=legacy`: Use legacy JUnit format

**Output Files**:
- `coverage.xml`: Coverage data
- `junit.xml`: Test results
- Terminal output with coverage summary

### Stage 8: Upload Coverage to Codecov

```yaml
- name: Upload coverage to Codecov
  uses: codecov/codecov-action@v5
  with:
    token: ${{ secrets.CODECOV_TOKEN }}
    files: ./coverage.xml
    flags: python-${{ matrix.python-version }}
    name: codecov-python-${{ matrix.python-version }}
    fail_ci_if_error: false
    verbose: true
```

**Purpose**: Send coverage data to Codecov

**Parameters**:
- `token`: Codecov authentication token (from secrets)
- `files`: Coverage report file
- `flags`: Tag this upload (e.g., python-3.10)
- `name`: Unique name for this upload
- `fail_ci_if_error: false`: Don't fail if upload has issues
- `verbose: true`: Show detailed upload logs

**Uploads**: 3 total (one per Python version)

### Stage 9: Upload Test Results to Codecov

```yaml
- name: Upload test results to Codecov
  if: ${{ !cancelled() }}
  uses: codecov/codecov-action@v5
  with:
    token: ${{ secrets.CODECOV_TOKEN }}
    files: ./junit.xml
    flags: python-${{ matrix.python-version }}
    name: test-results-python-${{ matrix.python-version }}
    report_type: test_results
    verbose: true
```

**Purpose**: Send test results to Codecov Test Analytics

**Key Differences from Coverage Upload**:
- `files: ./junit.xml`: JUnit XML file (not coverage.xml)
- `report_type: test_results`: Specifies test results (not coverage)
- `if: ${{ !cancelled() }}`: Runs even if previous steps failed

**Uploads**: 3 total (one per Python version)

### Stage 10: Upload Artifacts

```yaml
- name: Upload coverage reports as artifact
  uses: actions/upload-artifact@v4
  with:
    name: coverage-report-${{ matrix.python-version }}
    path: |
      coverage.xml
      htmlcov/
    retention-days: 30
```

**Purpose**: Store coverage reports for download

**Artifacts**:
- `coverage.xml`: XML coverage report
- `htmlcov/`: HTML coverage report directory

**Retention**: 30 days

**Access**: Download from GitHub Actions UI

## Matrix Testing

### What is Matrix Testing?

Run the same pipeline with different configurations in parallel.

### Our Matrix

```yaml
strategy:
  matrix:
    python-version: ['3.10', '3.11', '3.12']
```

**Result**: 3 parallel jobs

### Execution

Each Python version runs independently:

```
┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│ Python 3.10 │  │ Python 3.11 │  │ Python 3.12 │
│             │  │             │  │             │
│ ✓ Checkout  │  │ ✓ Checkout  │  │ ✓ Checkout  │
│ ✓ Setup     │  │ ✓ Setup     │  │ ✓ Setup     │
│ ✓ Install   │  │ ✓ Install   │  │ ✓ Install   │
│ ✓ Lint      │  │ ✓ Lint      │  │ ✓ Lint      │
│ ✓ Format    │  │ ✓ Format    │  │ ✓ Format    │
│ ✓ Type      │  │ ✓ Type      │  │ ✓ Type      │
│ ✓ Test      │  │ ✓ Test      │  │ ✓ Test      │
│ ✓ Upload    │  │ ✓ Upload    │  │ ✓ Upload    │
└─────────────┘  └─────────────┘  └─────────────┘
```

### Benefits

1. **Version Compatibility**: Ensure code works on all supported versions
2. **Parallel Execution**: Faster overall pipeline (3x speedup)
3. **Early Detection**: Catch version-specific bugs
4. **Coverage Tracking**: Separate coverage per version

### Viewing Matrix Results

In GitHub Actions UI:
- See all 3 jobs listed
- Each shows individual status
- Click to see logs for specific version
- Overall pipeline succeeds only if all 3 pass

## Code Quality Checks

### 1. Linting (flake8)

**What it Checks**:
- Syntax errors
- Undefined variables
- Unused imports
- Code complexity
- Style violations

**Configuration**:
- Max complexity: 10
- Max line length: 127
- Critical errors fail pipeline
- Style warnings don't fail

### 2. Formatting (black)

**What it Checks**:
- Code is formatted according to Black standard
- Consistent style across codebase

**Standard**:
- 88 character line length
- Double quotes for strings
- Specific indentation rules

**Fixing**:
```bash
black src tests
```

### 3. Type Checking (mypy)

**What it Checks**:
- Type hint consistency
- Type errors
- Return type matching

**Behavior**:
- Warnings only (doesn't fail pipeline)
- Encourages gradual type hint adoption

## Coverage and Test Reporting

### Coverage Upload

**Format**: XML (coverage.xml)
**Destination**: Codecov Coverage Analytics
**Frequency**: Every push and PR
**Flags**: Separate by Python version

### Test Results Upload

**Format**: JUnit XML (junit.xml)
**Destination**: Codecov Test Analytics
**Frequency**: Every push and PR
**Flags**: Separate by Python version

### Total Uploads Per Pipeline Run

- 3 coverage uploads (one per Python version)
- 3 test result uploads (one per Python version)
- **Total: 6 uploads**

### Upload Configuration

**Both uploads use**:
- Same token (CODECOV_TOKEN)
- Same flags (python-$version)
- Same action (codecov-action@v5)
- Different files (coverage.xml vs junit.xml)
- Different report types

## Secrets Management

### Required Secrets

1. **CODECOV_TOKEN**
   - Purpose: Authenticate with Codecov
   - Type: Global organization token
   - Value: `a9881dbe-674c-4b4c-a6be-d56525df0138`
   - Location: Repository Settings > Secrets

### Setting Secrets

**Via GitHub UI**:
1. Go to repository Settings
2. Click Secrets and variables > Actions
3. Click "New repository secret"
4. Name: `CODECOV_TOKEN`
5. Value: [paste token]
6. Click "Add secret"

**Via GitHub CLI**:
```bash
echo "a9881dbe-674c-4b4c-a6be-d56525df0138" | gh secret set CODECOV_TOKEN
```

**Verify**:
```bash
gh secret list
```

### Using Secrets in Workflow

```yaml
- name: Upload coverage to Codecov
  uses: codecov/codecov-action@v5
  with:
    token: ${{ secrets.CODECOV_TOKEN }}
```

**Security**:
- Secrets are encrypted
- Not visible in logs
- Only accessible during workflow execution
- Can't be retrieved after setting

## Artifacts

### What Are Artifacts?

Files generated during pipeline that you can download later.

### Our Artifacts

**Name**: `coverage-report-{python-version}`

**Contents**:
- `coverage.xml`: XML coverage report
- `htmlcov/`: HTML coverage report directory

**Retention**: 30 days

### Downloading Artifacts

1. Go to GitHub Actions
2. Click on a workflow run
3. Scroll to "Artifacts" section
4. Click artifact name to download

### Viewing HTML Coverage

```bash
# Download artifact
# Extract zip file
cd coverage-report-3.11

# Open HTML report
open htmlcov/index.html
```

## Status Checks

### GitHub Status Checks

Each pipeline run creates status checks on commits and PRs:

✓ **test (3.10)** - All checks passed
✓ **test (3.11)** - All checks passed
✓ **test (3.12)** - All checks passed

### Codecov Status Checks

Codecov adds additional checks:

✓ **codecov/project** - Coverage meets target
✓ **codecov/patch** - New code coverage meets target

### Branch Protection

**Recommended Settings**:

```yaml
# In GitHub Settings > Branches > Branch protection rules
Require status checks to pass:
  ✓ test (3.10)
  ✓ test (3.11)
  ✓ test (3.12)
  ✓ codecov/project
  ✓ codecov/patch
```

**Effect**: Cannot merge PR unless all checks pass

## Troubleshooting CI Failures

### Viewing Logs

1. Go to GitHub Actions tab
2. Click failed workflow run
3. Click failed job (e.g., "test (3.11)")
4. Expand failed step
5. Read error message

### Common Failures

#### 1. Lint Failures

**Error**:
```
src/calculator.py:10:1: F821 undefined name 'x'
```

**Fix**:
```bash
# Run locally
flake8 src

# Fix issues
# Re-run
flake8 src
```

#### 2. Format Failures

**Error**:
```
would reformat src/calculator.py
1 file would be reformatted
```

**Fix**:
```bash
# Format code
black src tests

# Commit
git add .
git commit -m "style: format code with black"
```

#### 3. Test Failures

**Error**:
```
FAILED tests/test_example.py::test_add - AssertionError: assert 4 == 5
```

**Fix**:
```bash
# Run tests locally
pytest -v

# Fix the bug
# Re-run tests
pytest -v
```

#### 4. Coverage Upload Failures

**Error**:
```
Error uploading coverage: 403 Forbidden
```

**Solutions**:
1. Check CODECOV_TOKEN is set
2. Verify token is valid
3. Check token has correct permissions
4. Try regenerating token

#### 5. Import Errors in CI

**Error**:
```
ModuleNotFoundError: No module named 'src'
```

**Solutions**:
1. Ensure all dependencies in requirements-test.txt
2. Check import statements
3. Verify package structure
4. Check PYTHONPATH configuration

### Debugging Tips

**1. Reproduce Locally**:
```bash
# Activate venv
source venv/bin/activate

# Run exact CI commands
python -m pip install --upgrade pip
pip install -r requirements-test.txt
flake8 src
black --check src tests
mypy src
pytest --cov=src
```

**2. Check Python Version**:
```bash
# Ensure local version matches CI
python --version
```

**3. Check Dependencies**:
```bash
# Verify all deps installed
pip list
```

**4. Enable Debug Logging**:
```yaml
- name: Run tests
  run: |
    pytest -vv --tb=long
  env:
    PYTEST_DEBUG: 1
```

## Best Practices

### 1. Keep Pipeline Fast

- Use pip caching
- Run tests in parallel where possible
- Avoid unnecessary steps
- Cache dependencies

**Current Speed**: ~2-3 minutes per Python version (parallel)

### 2. Fail Fast

```yaml
- name: Lint with flake8
  run: |
    # Critical errors fail immediately
    flake8 src --count --select=E9,F63,F7,F82
```

**Benefit**: Don't waste time if code has syntax errors

### 3. Use Matrix for Multi-Version Testing

```yaml
strategy:
  matrix:
    python-version: ['3.10', '3.11', '3.12']
```

**Benefit**: Parallel execution, version compatibility

### 4. Separate Coverage and Test Uploads

```yaml
# Coverage upload
report_type: coverage  # default

# Test results upload
report_type: test_results
```

**Benefit**: Both Coverage Analytics and Test Analytics

### 5. Always Upload Test Results

```yaml
- name: Upload test results
  if: ${{ !cancelled() }}  # Run even if tests failed
```

**Benefit**: See test results even when tests fail

### 6. Store Artifacts

```yaml
- name: Upload coverage reports
  uses: actions/upload-artifact@v4
```

**Benefit**: Download and review locally

### 7. Use Verbose Logging for Uploads

```yaml
with:
  verbose: true
```

**Benefit**: Easier debugging of upload issues

### 8. Don't Fail on Upload Errors

```yaml
with:
  fail_ci_if_error: false
```

**Benefit**: Tests can still pass if Codecov has issues

## Advanced Features

### 1. Conditional Steps

Run steps only in certain conditions:

```yaml
- name: Deploy to production
  if: github.ref == 'refs/heads/main' && github.event_name == 'push'
  run: |
    # Deploy code
```

### 2. Manual Triggers

Add manual workflow dispatch:

```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environment to test'
        required: true
        default: 'staging'
```

### 3. Scheduled Runs

Run pipeline on schedule:

```yaml
on:
  schedule:
    - cron: '0 0 * * 0'  # Every Sunday at midnight
```

### 4. Concurrency Control

Prevent multiple runs:

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

### 5. Reusable Workflows

Extract common steps:

```yaml
# .github/workflows/reusable-test.yml
on:
  workflow_call:
    inputs:
      python-version:
        required: true
        type: string

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      # Test steps here
```

### 6. Environment Variables

Set global env vars:

```yaml
env:
  PYTHONUNBUFFERED: 1
  FORCE_COLOR: 1

jobs:
  test:
    steps:
      - run: pytest
```

### 7. Custom Actions

Create project-specific actions:

```yaml
- name: Custom setup
  uses: ./.github/actions/setup-project
```

## Monitoring and Metrics

### GitHub Actions Insights

View metrics:
1. Go to repository
2. Click "Insights" tab
3. Click "Actions" in sidebar
4. See:
   - Workflow runs over time
   - Success/failure rates
   - Run durations
   - Most used workflows

### Codecov Dashboard

Monitor coverage:
- **URL**: https://app.codecov.io/gh/AXD-dev01/test-coverage-demo
- View coverage trends
- See flag-specific coverage
- Review test analytics

### Pipeline Status Badge

Add to README.md:

```markdown
![CI](https://github.com/AXD-dev01/test-coverage-demo/workflows/Python%20CI%20with%20Coverage/badge.svg)
```

### Notifications

Get notified of failures:
1. Go to GitHub Settings > Notifications
2. Enable "Actions" notifications
3. Choose notification method (email, web, mobile)

## Performance Optimization

### 1. Use Pip Caching

```yaml
- name: Set up Python
  uses: actions/setup-python@v5
  with:
    cache: 'pip'
```

**Speedup**: 30-60 seconds per run

### 2. Parallel Testing

```yaml
- name: Run tests
  run: |
    pytest -n auto  # Use all CPU cores
```

**Speedup**: 2-4x for large test suites

### 3. Minimal Checkout

```yaml
- uses: actions/checkout@v4
  with:
    fetch-depth: 1  # Shallow clone (if full history not needed)
```

**Speedup**: 5-10 seconds for large repos

### 4. Fail Fast Strategy

```yaml
strategy:
  fail-fast: true  # Stop all jobs if one fails
  matrix:
    python-version: ['3.10', '3.11', '3.12']
```

**Benefit**: Don't waste resources on doomed runs

## Related Documentation

- [CODECOV.md](CODECOV.md): Codecov integration guide
- [TESTING.md](TESTING.md): Testing guide
- [DEVELOPMENT.md](DEVELOPMENT.md): Development setup
- [CONTRIBUTING.md](CONTRIBUTING.md): Contribution guidelines

## External Resources

### GitHub Actions

- **Docs**: https://docs.github.com/en/actions
- **Marketplace**: https://github.com/marketplace?type=actions
- **Workflow Syntax**: https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions

### Actions We Use

- **checkout**: https://github.com/actions/checkout
- **setup-python**: https://github.com/actions/setup-python
- **codecov-action**: https://github.com/codecov/codecov-action
- **upload-artifact**: https://github.com/actions/upload-artifact

### Best Practices

- **Security**: https://docs.github.com/en/actions/security-guides
- **Optimization**: https://docs.github.com/en/actions/guides/about-continuous-integration

---

**Last Updated**: December 31, 2025
**Pipeline Status**: All checks passing
**Python Versions**: 3.10, 3.11, 3.12
**Average Run Time**: 2-3 minutes
