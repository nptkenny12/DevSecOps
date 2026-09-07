# Lab 5.3 — Submission

## Task 1: GitHub Actions SAST + DAST Pipeline

### Workflow file

The workflow is defined in `.github/workflows/lab5-sast-dast.yml` and contains two jobs: `sast` for Semgrep and `dast` for OWASP ZAP.

### Successful workflow run

- [Successful SAST and DAST workflow run](https://github.com/nptkenny12/DevSecOps/actions/runs/34141011037)
- `SAST — Semgrep`: Success
- `DAST — OWASP ZAP`: Success

### SAST job

#### Triggers (`on:`)

The workflow runs on pushes to `main`, pull requests targeting `main`, and manual `workflow_dispatch` runs. 
Running the checks in CI ensures that SAST and DAST are repeated consistently during integration, even when a developer forgets to run the local scans.

#### Clone Juice Shop source (pinned to `v20.0.0`)

The workflow clones Juice Shop and checks out `v20.0.0`, matching the version used in Lab 5.1 and the Docker image used in Lab 5.2. Pinning the version makes the scan reproducible and prevents changes on the upstream `main` branch from changing the results unexpectedly.

#### Run Semgrep (JSON report)

Semgrep uses the `p/owasp-top-ten`, `p/javascript`, and `p/secrets` rulesets. It writes a machine-readable JSON report and a human-readable summary. The scan is allowed to report findings without stopping the pipeline because Juice Shop intentionally contains vulnerable examples; the workflow still checks that the report was actually produced.

#### Upload SAST reports

The workflow uploads `semgrep.json` and `semgrep.txt` as the `lab5-sast-reports` artifact and keeps them for 30 days. `if: always()` ensures the reports remain available for investigation even when a previous scan step reports findings.

### DAST job

#### Start Juice Shop / Wait for Juice Shop to be ready

The job starts the pinned Juice Shop image on the dedicated `lab5-net` Docker network so that ZAP can reach it by the hostname `juice-shop`. The health-check loop repeatedly requests the application until it responds or the startup timeout is reached.

#### Run ZAP baseline scan

`zap-baseline.py` performs an unauthenticated passive baseline scan against the running application. Exit code `2` is accepted because it means ZAP found alerts; exit code `1` is treated as a scan error. The JSON and HTML reports must still be created.

#### Run ZAP authenticated scan

The Automation Framework configuration in `labs/lab5/scripts/zap-auth.yaml` logs in with the configured Juice Shop test user, crawls authenticated routes, waits for passive scanning, and runs the active scan. `_JAVA_OPTIONS="-Xmx512m"` limits ZAP's heap usage so the GitHub-hosted runner does not run out of memory.

#### Compare baseline vs authenticated reports

`compare_zap.sh` reads both JSON reports and writes `zap-comparison.txt` with severity counts for unauthenticated and authenticated scans. 
This automates the comparison and supports the alert-ratio and auth-only analysis documented in Lab 5.2.

#### Stop Juice Shop

The cleanup step uses `if: always()` so the container and Docker network are removed even if a scan or report-validation step fails. 
This prevents leftover containers and resources on the GitHub-hosted runner.

### Reflection

This CI pipeline complements the local Semgrep and ZAP scans from Labs 5.1 and 5.2 by running the same controls automatically on pushes and pull requests. I would still run scans locally for faster feedback while developing or debugging a finding, then use CI as the repeatable integration check before merging.

