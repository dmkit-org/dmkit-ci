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

### Release

Automates versioning, changelog generation, and GitHub releases using [semantic-release](https://semantic-release.gitbook.io/).

**Version bumps from commit types:**

- `fix:` → patch (1.0.0 → 1.0.1)
- `feat:` → minor (1.0.0 → 1.1.0)
- `feat!:` or `BREAKING CHANGE:` → major (1.0.0 → 2.0.0)

**What it does:** Creates a git tag, GitHub Release with generated notes, and updates `CHANGELOG.md` in the calling repo.

**Required:** A token with `contents: write` permission.

**Usage:**

```yaml
# .github/workflows/release.yml
name: Release
on:
  push:
    branches: [main]

jobs:
  release:
    uses: dmkit-org/dmkit-ci/.github/workflows/release.yml@main
    secrets:
      token: ${{ secrets.GITHUB_TOKEN }}
```

## Advisory Linting

Both linting workflows use `continue-on-error: true`. Results are reported but never block the calling workflow.
