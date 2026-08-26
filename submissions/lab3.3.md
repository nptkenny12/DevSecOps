# Lab 3.3 — Submission

## Task: Gitleaks CI Scan

### Workflow file

```yaml
name: Gitleaks Scan

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  gitleaks:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Run Gitleaks
        uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### Successful workflow run

- Direct link to the successful GitHub Actions run: https://github.com/nptkenny12/DevSecOps/actions/runs/32956023213/job/98137740632
- Workflow name: **Gitleaks Scan**
- Job status: **Success**
- The `gitleaks` job completed successfully with all steps passing.

### Job step explanation

#### Triggers (`on:`)

The workflow starts when code is pushed to `main` or when a pull request targets `main`. It also supports `workflow_dispatch`, which allows the workflow to be started manually from the GitHub Actions page. Scanning both pushes and pull requests provides protection before and after changes are merged.

#### Job: `gitleaks` / `runs-on: ubuntu-latest`

The `gitleaks` job performs the repository secret scan. It runs on a GitHub-hosted Ubuntu runner so the scan uses a clean and consistent environment rather than depending on a developer's local machine.

#### Step: Checkout repository

`actions/checkout@v4` downloads the repository contents to the GitHub Actions runner. `fetch-depth: 0` retrieves the complete Git history, which is important because Gitleaks must scan previous commits as well as the current files.

#### Step: Run Gitleaks

`gitleaks/gitleaks-action@v2` scans the repository for exposed secrets and fails the workflow when a secret is detected. `GITHUB_TOKEN` is the automatically provided GitHub token that allows the action to interact with GitHub and report the workflow result.

### Workflow result

The workflow run triggered by the pull request completed successfully. The `gitleaks` job, repository checkout, Gitleaks scan, and final job completion steps all showed a green success status. GitHub displayed one Node.js deprecation warning, but it was only an annotation and did not cause the workflow to fail.

### Reflection

CI scanning is still necessary even when developers use a Gitleaks pre-commit hook because local hooks can be skipped, bypassed with `--no-verify`, or missing on a developer's machine. CI provides a centralized second line of defense that scans changes on GitHub before they are merged.
