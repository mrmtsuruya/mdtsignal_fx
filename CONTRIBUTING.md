# CONTRIBUTING

## Title
mdtsignal_fx — Contribution Guidelines

## Purpose
This document provides the guidelines and standards that all contributors must follow when working on the mdtsignal_fx platform. It ensures consistency, code quality, and a productive collaboration experience.

## Status
Active

## Version
0.1.0

## Last Updated
2026-06-25

## Author
mrmtsuruya

---

## Table of Contents

1. [Code of Conduct](#code-of-conduct)
2. [Getting Started](#getting-started)
3. [Branch Strategy](#branch-strategy)
4. [Commit Conventions](#commit-conventions)
5. [Pull Request Process](#pull-request-process)
6. [Code Style](#code-style)
7. [Testing Requirements](#testing-requirements)
8. [Documentation Standards](#documentation-standards)
9. [Security Policy](#security-policy)

---

## Code of Conduct

All contributors are expected to maintain a respectful and professional environment. Harassment, discrimination, or disruptive behavior of any kind will not be tolerated.

---

## Getting Started

1. Fork the repository
2. Clone your fork locally
3. Create a new branch from `main` following the branch naming conventions below
4. Make your changes following the code style and testing requirements
5. Open a Pull Request against `main`

---

## Branch Strategy

| Branch | Purpose |
|---|---|
| `main` | Stable, production-ready code |
| `develop` | Integration branch for features |
| `feature/<name>` | New features |
| `fix/<name>` | Bug fixes |
| `docs/<name>` | Documentation updates |
| `chore/<name>` | Maintenance and tooling changes |

---

## Commit Conventions

This project follows the [Conventional Commits](https://www.conventionalcommits.org/) specification.

**Format:** `<type>(<scope>): <subject>`

| Type | Usage |
|---|---|
| `feat` | New feature |
| `fix` | Bug fix |
| `docs` | Documentation change |
| `chore` | Build process, dependency update |
| `test` | Adding or updating tests |
| `refactor` | Code refactoring |
| `perf` | Performance improvement |
| `ci` | CI/CD configuration |

**Example:**
```
feat(hermes): add webhook signal receiver endpoint
fix(mt5): correct order size calculation for XAUUSD
docs(api): update signal payload schema
```

---

## Pull Request Process

1. Ensure all tests pass before opening a PR
2. Fill in the PR template completely
3. Link any related issues using `Closes #<issue_number>`
4. Request at least one review from a maintainer
5. Squash commits before merging, unless a merge commit is preferred by the maintainer

---

## Code Style

### Python
- Follow [PEP 8](https://peps.python.org/pep-0008/)
- Use type hints for all function signatures
- Maximum line length: 100 characters
- Formatter: `black`
- Linter: `ruff`

### MQL5
- Follow MetaQuotes MQL5 style guidelines
- Comment all non-obvious logic blocks

### Pine Script
- Use descriptive variable names
- Comment strategy logic and entry/exit conditions

---

## Testing Requirements

- All new features must include unit tests
- Minimum code coverage: **80%**
- Integration tests required for all external interface changes
- Tests must pass in CI before merge is permitted

---

## Documentation Standards

Every new specification or architecture document must include the following header:

```markdown
## Title
## Purpose
## Status
## Version
## Last Updated
## Author
## Table of Contents
```

---

## Security Policy

- **Never** commit secrets, credentials, API keys, or tokens
- All sensitive configuration must use environment variables
- Report security vulnerabilities privately to the repository maintainer before public disclosure
- Dependencies must be reviewed against known vulnerability databases before addition
