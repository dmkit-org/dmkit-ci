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

### Backend CI

Builds, tests, and deploys the Go backend. Runs `make test` and `make lint` on every PR. On push to `main`, also builds the Docker image, pushes to ECR, and updates the ECS service.

**Inputs:** `aws-region`, `ecr-repository`, `ecs-cluster`, `ecs-service`, `role-to-assume`, `deploy`

**OIDC prerequisite:** IAM role ARN from `terraform output github_actions_backend_role_arn`

**Usage:**

```yaml
jobs:
  ci:
    uses: dmkit-org/dmkit-ci/.github/workflows/backend-ci.yml@main
    with:
      aws-region: us-east-1
      ecr-repository: dmkit-backend
      ecs-cluster: dmkit-dev
      ecs-service: dmkit-backend
      role-to-assume: ${{ vars.AWS_ROLE_ARN }}
      deploy: ${{ github.event_name == 'push' && github.ref == 'refs/heads/main' }}
```

### Frontend CI

Builds, lints, and deploys the Next.js frontend. Runs `npm run lint` and `npm run build` on every PR. On push to `main`, also syncs static assets to S3 and invalidates the CloudFront cache.

**Inputs:** `aws-region`, `s3-bucket`, `cloudfront-distribution-id`, `role-to-assume`, `deploy`

**OIDC prerequisite:** IAM role ARN from `terraform output github_actions_frontend_role_arn`

**Usage:**

```yaml
jobs:
  ci:
    uses: dmkit-org/dmkit-ci/.github/workflows/frontend-ci.yml@main
    with:
      aws-region: us-east-1
      s3-bucket: ${{ vars.S3_BUCKET }}
      cloudfront-distribution-id: ${{ vars.CLOUDFRONT_DISTRIBUTION_ID }}
      role-to-assume: ${{ vars.AWS_ROLE_ARN }}
      deploy: ${{ github.event_name == 'push' && github.ref == 'refs/heads/main' }}
```

### Infra CI

Runs Terraform plan on every PR. On push to `main`, also runs Terraform apply.

**Inputs:** `aws-region`, `working-directory` (default: `envs/dev/`), `role-to-assume`, `apply`

**OIDC prerequisite:** IAM role ARN from `terraform output github_actions_infra_role_arn`

**Usage:**

```yaml
jobs:
  ci:
    uses: dmkit-org/dmkit-ci/.github/workflows/infra-ci.yml@main
    with:
      aws-region: us-east-1
      role-to-assume: ${{ vars.AWS_ROLE_ARN }}
      apply: ${{ github.event_name == 'push' && github.ref == 'refs/heads/main' }}
```

## Advisory Linting

Both linting workflows use `continue-on-error: true`. Results are reported but never block the calling workflow.
