# DMKit CI

Shared CI/CD workflows and configuration for DMKit repositories.

## Available Workflows

### Commitlint

Validates PR titles and commit messages against [Conventional Commits](https://www.conventionalcommits.org/).

**Allowed types:** `feat`, `fix`, `chore`, `docs`, `refactor`, `test`, `ci`

**Usage:**

```yaml
# .github/workflows/lint.yml
name: Lint
on:
  pull_request:

jobs:
  commitlint:
    uses: dmkit-org/dmkit-ci/.github/workflows/commitlint.yml@main
```

### Markdownlint

Lints all markdown files in the calling repository.

**Usage:**

```yaml
# .github/workflows/lint.yml (add as a second job)
jobs:
  markdownlint:
    uses: dmkit-org/dmkit-ci/.github/workflows/markdownlint.yml@main
```

## Advisory Linting

Both linting workflows use `continue-on-error: true`. Results are reported but never block the calling workflow.
