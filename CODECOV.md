# Codecov Integration Guide

Complete guide to Codecov integration, configuration, and usage for the test-coverage-demo project.

## Table of Contents

- [What is Codecov](#what-is-codecov)
- [Why We Use Codecov](#why-we-use-codecov)
- [Quick Start](#quick-start)
- [Configuration](#configuration)
- [Coverage Badges](#coverage-badges)
- [Understanding Coverage Reports](#understanding-coverage-reports)
- [Using the Codecov Dashboard](#using-the-codecov-dashboard)
- [Coverage Analytics](#coverage-analytics)
- [Test Analytics](#test-analytics)
- [Flags and Multi-Version Testing](#flags-and-multi-version-testing)
- [Pull Request Workflow](#pull-request-workflow)
- [Troubleshooting](#troubleshooting)
- [Advanced Features](#advanced-features)
- [Links and Resources](#links-and-resources)

## What is Codecov

[Codecov](https://codecov.io) is a code coverage reporting service that helps teams:

- **Track test coverage** across commits and pull requests
- **Visualize coverage** with interactive reports and graphs
- **Enforce quality standards** with configurable coverage targets
- **Analyze trends** over time to improve code quality
- **Block PRs** that reduce coverage below acceptable thresholds

### Key Features We Use

1. **Coverage Analytics**: Track line, branch, and function coverage
2. **Test Analytics**: Monitor test execution and performance
3. **Multi-Version Flags**: Separate coverage for Python 3.10, 3.11, and 3.12
4. **PR Comments**: Automatic coverage reports on pull requests
5. **GitHub Integration**: Status checks that can block merges
6. **Global Organization Standards**: Consistent quality across all repositories

## Why We Use Codecov

### Benefits for This Project

1. **Quality Assurance**: Maintain 100% test coverage
2. **Regression Prevention**: Block PRs that reduce coverage
3. **Multi-Version Testing**: Ensure compatibility across Python versions
4. **Visibility**: See exactly which lines are/aren't tested
5. **Trend Analysis**: Track coverage improvements over time
6. **Team Collaboration**: Coverage reports on every PR

### What We Track

- **Line Coverage**: Which lines of code are executed by tests
- **Branch Coverage**: Which conditional branches are tested
- **Function Coverage**: Which functions are called by tests
- **Test Results**: Which tests pass/fail across Python versions

## Quick Start

### Viewing Coverage Locally

```bash
# Run tests with coverage
pytest --cov=src --cov-report=html

# Open HTML report
open htmlcov/index.html  # macOS
# or
xdg-open htmlcov/index.html  # Linux
```

### Viewing Coverage on Codecov

1. **Visit the dashboard**: https://app.codecov.io/gh/AXD-dev01/test-coverage-demo
2. **Click on a commit** to see coverage for that commit
3. **Click on a file** to see line-by-line coverage
4. **View graphs** to see coverage trends over time

### Coverage Badge

Add to your README:

```markdown
[![codecov](https://codecov.io/gh/AXD-dev01/test-coverage-demo/branch/main/graph/badge.svg)](https://codecov.io/gh/AXD-dev01/test-coverage-demo)
```

## Configuration

We use a two-tier configuration approach:

1. **Global Organization YAML**: Applies to all AXD-dev01 repositories
2. **Repository YAML**: Project-specific overrides and settings

### Repository Configuration

**File**: `codecov.yml` (in project root)

```yaml
# Coverage configuration
coverage:
  precision: 2  # Show 2 decimal places (e.g., 98.75%)
  round: down   # Round down for conservative estimates
  range: "70...100"  # Color coding: red (70%) to green (100%)

  # Status checks
  status:
    project:
      default:
        target: 80%           # Minimum 80% coverage required
        threshold: 2%         # Allow max 2% drop
        if_ci_failed: error   # Fail if CI fails
        informational: false  # Block PR if below target

    patch:
      default:
        target: 80%           # New code must have 80%+ coverage
        threshold: 2%
        if_ci_failed: error

# Comment on PRs after all 3 Python versions report
comment:
  after_n_builds: 3

# Flags for organizing coverage
flags:
  python-3.10:
    carryforward: true
  python-3.11:
    carryforward: true
  python-3.12:
    carryforward: true

# Ignore test files and build artifacts
ignore:
  - "tests/**/*"
  - "**/__pycache__/**/*"
  - "**/venv/**/*"
  - "**/htmlcov/**/*"

# Wait for CI before processing
codecov:
  max_report_age: off
  require_ci_to_pass: yes
```

### Global Organization Configuration

**Location**: https://app.codecov.io/account/gh/AXD-dev01/yaml

**Purpose**: Enforce organization-wide quality standards

**Key Settings**:
- **80% minimum** coverage for all projects
- **85% minimum** for new code (patches)
- **90% minimum** for critical paths (core, api, security)
- **Max 1% coverage drop** allowed
- **Comprehensive ignore patterns** for tests, build artifacts, configs
- **Professional PR comments** with coverage diff

**Note**: Repository `codecov.yml` can override global settings if needed.

## Coverage Badges

### Adding Badges to README

```markdown
# Coverage Badge
[![codecov](https://codecov.io/gh/AXD-dev01/test-coverage-demo/branch/main/graph/badge.svg)](https://codecov.io/gh/AXD-dev01/test-coverage-demo)

# Coverage Graph
[![codecov](https://codecov.io/gh/AXD-dev01/test-coverage-demo/branch/main/graphs/sunburst.svg)](https://codecov.io/gh/AXD-dev01/test-coverage-demo)

# Coverage Icicle
[![codecov](https://codecov.io/gh/AXD-dev01/test-coverage-demo/branch/main/graphs/icicle.svg)](https://codecov.io/gh/AXD-dev01/test-coverage-demo)
```

### Badge Styles

```markdown
# Default
![codecov](https://codecov.io/gh/AXD-dev01/test-coverage-demo/branch/main/graph/badge.svg)

# Flat
![codecov](https://codecov.io/gh/AXD-dev01/test-coverage-demo/branch/main/graph/badge.svg?style=flat)

# Flat-square
![codecov](https://codecov.io/gh/AXD-dev01/test-coverage-demo/branch/main/graph/badge.svg?style=flat-square)

# For the badge
![codecov](https://codecov.io/gh/AXD-dev01/test-coverage-demo/branch/main/graph/badge.svg?style=for-the-badge)
```

### Custom Token (for Private Repos)

```markdown
![codecov](https://codecov.io/gh/AXD-dev01/test-coverage-demo/branch/main/graph/badge.svg?token=YOUR_TOKEN_HERE)
```

## Understanding Coverage Reports

### Coverage Types

1. **Line Coverage**
   - **Green**: Line was executed by tests
   - **Red**: Line was NOT executed by tests
   - **Yellow**: Line was partially executed (some branches missed)

2. **Branch Coverage**
   - Tracks if/else, switch/case, and other conditional branches
   - Ensures both true and false paths are tested

3. **Function Coverage**
   - Tracks which functions are called during tests
   - Helps identify unused code

### Coverage Metrics

```
Coverage: 100.00%
Files: 2
Lines: 45
Hits: 45
Misses: 0
Partials: 0
Branches: 12
Branch Hits: 12
```

**Definitions**:
- **Hits**: Lines executed by tests
- **Misses**: Lines NOT executed by tests
- **Partials**: Lines partially executed (some branches missed)
- **Branches**: Total conditional branches
- **Branch Hits**: Branches executed by tests

### Reading the HTML Report

```bash
# Generate HTML report
pytest --cov=src --cov-report=html

# Open in browser
open htmlcov/index.html
```

**Report Features**:
- **File listing**: Overview of coverage per file
- **Line-by-line view**: Click files to see detailed coverage
- **Color coding**: Green (covered), red (missed), yellow (partial)
- **Coverage percentage**: Per file and overall

## Using the Codecov Dashboard

### Dashboard Overview

**URL**: https://app.codecov.io/gh/AXD-dev01/test-coverage-demo

**Main Sections**:

1. **Commits Tab**
   - View coverage for each commit
   - See coverage changes over time
   - Filter by branch

2. **Pulls Tab**
   - Coverage reports for open PRs
   - See coverage impact before merging
   - Compare base vs head coverage

3. **Flags Tab**
   - View coverage by flag (python-3.10, python-3.11, python-3.12)
   - Compare coverage across Python versions
   - Track version-specific issues

4. **Files Tab**
   - Browse project files
   - See coverage for each file
   - Click for line-by-line view

5. **Settings Tab**
   - Configure YAML settings
   - Manage tokens
   - Configure integrations

### Viewing Coverage for a Commit

1. Go to **Commits** tab
2. Click on a commit
3. See:
   - Overall coverage percentage
   - Coverage change from previous commit
   - Files changed with coverage impact
   - Line-by-line coverage for changed files

### Viewing Coverage for a File

1. Go to **Files** tab
2. Navigate to file (e.g., `src/calculator.py`)
3. See:
   - Green highlights: Lines covered by tests
   - Red highlights: Lines NOT covered
   - Yellow highlights: Partially covered lines
   - Coverage percentage for that file

### Coverage Graphs

**Sunburst Graph**: Visual representation of coverage hierarchy
```
https://codecov.io/gh/AXD-dev01/test-coverage-demo/branch/main/graphs/sunburst.svg
```

**Icicle Graph**: Alternative visualization
```
https://codecov.io/gh/AXD-dev01/test-coverage-demo/branch/main/graphs/icicle.svg
```

**Tree Graph**: File tree with coverage
```
https://codecov.io/gh/AXD-dev01/test-coverage-demo/branch/main/graphs/tree.svg
```

## Coverage Analytics

### What is Coverage Analytics?

Coverage Analytics tracks:
- **Line coverage**: Which lines are executed
- **Branch coverage**: Which conditional paths are tested
- **Function coverage**: Which functions are called
- **File coverage**: Per-file coverage percentages

### How We Use It

**In CI/CD**:
```yaml
- name: Upload coverage to Codecov
  uses: codecov/codecov-action@v5
  with:
    token: ${{ secrets.CODECOV_TOKEN }}
    files: ./coverage.xml
    flags: python-${{ matrix.python-version }}
    fail_ci_if_error: false
    verbose: true
```

**Coverage File**: `coverage.xml` (generated by pytest-cov)

**Flags**: Separate coverage for each Python version
- `python-3.10`
- `python-3.11`
- `python-3.12`

### Coverage Trends

View coverage over time:
1. Go to Codecov dashboard
2. Select **Graphs** tab
3. See:
   - Coverage percentage over commits
   - Trend line (improving/declining)
   - Coverage changes per commit

## Test Analytics

### What is Test Analytics?

Test Analytics tracks:
- **Test execution**: Which tests run
- **Test results**: Pass/fail status
- **Test duration**: How long tests take
- **Test trends**: Flaky tests, failures over time

### How We Use It

**In CI/CD**:
```yaml
- name: Upload test results to Codecov
  uses: codecov/codecov-action@v5
  with:
    token: ${{ secrets.CODECOV_TOKEN }}
    files: ./junit.xml
    flags: python-${{ matrix.python-version }}
    report_type: test_results
    verbose: true
```

**Test Results File**: `junit.xml` (generated by pytest with `--junitxml`)

**What We Track**:
- 7 tests across 3 Python versions = 21 test executions
- Test names and outcomes
- Execution time per test
- Historical test performance

### Viewing Test Results

1. Go to Codecov dashboard
2. Navigate to **Tests** tab
3. See:
   - List of all tests
   - Pass/fail status
   - Execution time
   - Trends and flakiness detection

## Flags and Multi-Version Testing

### What are Flags?

Flags organize coverage into logical groups. We use flags to separate coverage by Python version.

### Our Flags

```yaml
flags:
  python-3.10:
    carryforward: true
  python-3.11:
    carryforward: true
  python-3.12:
    carryforward: true
```

**Carryforward**: If a flag doesn't report in a commit (e.g., only Python 3.11 ran), Codecov will use the previous report for that flag.

### Uploading with Flags

```yaml
- name: Upload coverage to Codecov
  uses: codecov/codecov-action@v5
  with:
    flags: python-${{ matrix.python-version }}
```

This creates separate coverage reports:
- `python-3.10` flag: Coverage from Python 3.10 tests
- `python-3.11` flag: Coverage from Python 3.11 tests
- `python-3.12` flag: Coverage from Python 3.12 tests

### Viewing Flag Coverage

1. Go to **Flags** tab in Codecov dashboard
2. Select a flag (e.g., `python-3.10`)
3. See coverage specific to that Python version
4. Compare coverage across versions

### Why This Matters

- **Version-specific bugs**: Some code paths may only execute in certain Python versions
- **Compatibility testing**: Ensure code works across all supported versions
- **Trend tracking**: Monitor coverage improvements per version

## Pull Request Workflow

### How It Works

1. **Developer creates PR**
2. **GitHub Actions runs**:
   - Tests on Python 3.10, 3.11, 3.12
   - Generates coverage reports
   - Uploads to Codecov (3 coverage + 3 test result uploads = 6 total)
3. **Codecov processes reports**:
   - Compares PR coverage to base branch
   - Calculates coverage change
   - Waits for all 3 builds (configured with `after_n_builds: 3`)
4. **Codecov comments on PR**:
   - Shows coverage percentage
   - Highlights coverage changes
   - Shows which files changed coverage
5. **GitHub status check**:
   - Pass: Coverage meets requirements
   - Fail: Coverage below threshold

### PR Comment Example

```markdown
## Codecov Report
Coverage: 98.50% (+0.50%) compared to base

| Files | Coverage Δ | Complexity Δ |
|-------|------------|---------------|
| src/calculator.py | 95.00% (+2.00%) | 15 (+1) |

Flags:
- python-3.10: 98.50%
- python-3.11: 98.50%
- python-3.12: 98.50%

Continue to review full report at Codecov.
```

### Status Check Requirements

**Configured in `codecov.yml`**:

```yaml
status:
  project:
    default:
      target: 80%           # Project must have 80%+ coverage
      threshold: 2%         # Allow max 2% drop
      informational: false  # Block merge if below target

  patch:
    default:
      target: 80%           # New code must have 80%+ coverage
      threshold: 2%
```

**Status Check Results**:
- **Success**: Coverage meets requirements
- **Failure**: Coverage below target or dropped more than threshold

### Handling Coverage Drops

If PR reduces coverage:

1. **Add more tests** to cover new code
2. **Check for dead code** that can be removed
3. **Review coverage report** to see what's missing
4. **Add tests for edge cases** and error paths

### Configuring PR Comments

```yaml
comment:
  layout: "header, diff, flags, files, footer"
  behavior: default          # Update same comment
  require_changes: false     # Always comment
  after_n_builds: 3          # Wait for all Python versions
```

## Troubleshooting

### Issue: Coverage Not Uploading

**Symptoms**: No coverage on Codecov dashboard

**Solutions**:

1. **Check GitHub Actions logs**:
   ```bash
   gh run view
   ```

2. **Verify CODECOV_TOKEN is set**:
   ```bash
   gh secret list
   ```

3. **Check coverage.xml was generated**:
   ```yaml
   - name: Upload coverage artifacts
     uses: actions/upload-artifact@v3
     with:
       name: coverage-reports
       path: coverage.xml
   ```

4. **Verify codecov.yml is valid**:
   ```bash
   curl --data-binary @codecov.yml https://codecov.io/validate
   ```

### Issue: PR Comments Not Appearing

**Symptoms**: No Codecov comment on PR

**Solutions**:

1. **Check `after_n_builds` setting**:
   - Set to `3` means Codecov waits for all 3 Python versions
   - If only 2 builds complete, no comment appears
   - Solution: Wait for all builds or reduce `after_n_builds`

2. **Verify all uploads succeeded**:
   - Check GitHub Actions logs
   - Look for "Upload coverage to Codecov" steps
   - Ensure all 3 succeeded

3. **Check repository settings**:
   - Go to Codecov dashboard > Settings
   - Verify "PR Comments" is enabled

### Issue: Coverage Shows 0%

**Symptoms**: Codecov shows 0% coverage

**Solutions**:

1. **Check coverage.xml has data**:
   ```bash
   cat coverage.xml
   ```

2. **Verify source paths**:
   - Coverage report paths must match repository structure
   - Check `coverage.xml` for correct file paths

3. **Check ignore patterns**:
   - Ensure you're not ignoring source code
   - Review `codecov.yml` ignore section

### Issue: "Upload exceeds max age"

**Symptoms**: Upload fails with age error

**Solution**:
```yaml
codecov:
  max_report_age: off  # Disable age check
```

### Issue: Multiple Builds Not Showing

**Symptoms**: Only seeing coverage from one Python version

**Solutions**:

1. **Check flags are different**:
   ```yaml
   flags: python-${{ matrix.python-version }}
   ```

2. **Verify matrix is working**:
   ```yaml
   strategy:
     matrix:
       python-version: ["3.10", "3.11", "3.12"]
   ```

3. **Check all jobs ran**:
   - View GitHub Actions workflow
   - Ensure all 3 Python versions executed

### Issue: Status Check Always Failing

**Symptoms**: PR status check fails even with good coverage

**Solutions**:

1. **Check threshold settings**:
   ```yaml
   status:
     project:
       default:
         threshold: 2%  # Too strict? Increase to 5%
   ```

2. **Check target is realistic**:
   ```yaml
   target: 80%  # Is this achievable for your code?
   ```

3. **Review coverage drop**:
   - Go to Codecov PR page
   - See which files dropped coverage
   - Add tests for those files

### Getting Help

1. **Codecov Documentation**: https://docs.codecov.com/
2. **GitHub Issues**: https://github.com/codecov/feedback/issues
3. **Community Forum**: https://community.codecov.com/
4. **Support Email**: support@codecov.io

## Advanced Features

### Component Management (for Monorepos)

Define components in `codecov.yml`:

```yaml
component_management:
  individual_components:
    - component_id: api
      name: API Layer
      paths:
        - src/api/**

    - component_id: database
      name: Database Layer
      paths:
        - src/database/**

    - component_id: frontend
      name: Frontend
      paths:
        - src/frontend/**
```

**Benefits**:
- Track coverage per component
- Set different targets for different components
- See component-specific trends

### Carryforward Flags

```yaml
flags:
  python-3.10:
    carryforward: true
```

**Purpose**: If a flag doesn't report (e.g., Python 3.10 tests didn't run), use the previous report for that flag.

**Use Cases**:
- Partial CI runs (only some Python versions)
- Flaky CI systems
- Development commits without full CI

### Path Fixes

If coverage paths don't match repository structure:

```yaml
fixes:
  - "before/path::after/path"
  - "src/::/"  # Remove src/ prefix
```

### Codecov YAML Validation

Validate your `codecov.yml` before committing:

```bash
# Validate local file
curl --data-binary @codecov.yml https://codecov.io/validate

# Validate from URL
curl -X POST --data-binary @codecov.yml https://codecov.io/validate
```

### Slack Notifications

Add to `codecov.yml`:

```yaml
slack:
  url: "secret:SLACK_WEBHOOK_URL"
  threshold: 1%
  only_pulls: true
  branches:
    - main
  message: "Coverage changed by {{changed}} for {{owner/repo}}"
  attachments: "sunburst, diff"
```

Then add GitHub secret:
```bash
gh secret set SLACK_WEBHOOK_URL
```

### Bundle Analysis (for JavaScript/TypeScript)

Track bundle sizes to prevent performance regressions:

```yaml
# In GitHub Actions
- name: Upload bundle stats
  uses: codecov/codecov-action@v5
  with:
    plugin: bundler
    token: ${{ secrets.CODECOV_TOKEN }}
```

**Supports**: Webpack, Rollup, Vite, Next.js, and more

## Links and Resources

### This Project

- **Repository**: https://github.com/AXD-dev01/test-coverage-demo
- **Codecov Dashboard**: https://app.codecov.io/gh/AXD-dev01/test-coverage-demo
- **GitHub Actions**: https://github.com/AXD-dev01/test-coverage-demo/actions

### Organization Settings

- **Organization Dashboard**: https://app.codecov.io/account/gh/AXD-dev01
- **Global YAML**: https://app.codecov.io/account/gh/AXD-dev01/yaml
- **Global Token**: https://app.codecov.io/account/gh/AXD-dev01/org-upload-token

### Codecov Documentation

- **Main Docs**: https://docs.codecov.com/
- **YAML Reference**: https://docs.codecov.com/docs/codecovyml-reference
- **Quick Start**: https://docs.codecov.com/docs/quick-start
- **GitHub Actions**: https://docs.codecov.com/docs/github-actions
- **Flags**: https://docs.codecov.com/docs/flags
- **Components**: https://docs.codecov.com/docs/components
- **Status Checks**: https://docs.codecov.com/docs/commit-status

### Tools

- **YAML Validator**: https://codecov.io/validate
- **Browser Extension**: https://github.com/codecov/codecov-browser-extension
- **VS Code Extension**: Search "Codecov" in VS Code marketplace

### Support

- **GitHub Issues**: https://github.com/codecov/feedback/issues
- **Community Forum**: https://community.codecov.com/
- **Support Email**: support@codecov.io
- **Status Page**: https://status.codecov.io/

### Related Documentation

- [TESTING.md](TESTING.md): Testing guide
- [CI_CD.md](CI_CD.md): CI/CD pipeline documentation
- [DEVELOPMENT.md](DEVELOPMENT.md): Development setup
- [CONTRIBUTING.md](CONTRIBUTING.md): Contribution guidelines

---

**Last Updated**: December 31, 2025
**Coverage Status**: 100%
**Organization**: AXD-dev01
