# Lab 4.2 — Submission

## Task 1: Trivy Comparison

### Side-by-side counts
| Severity | Grype | Trivy | Δ |
|----------|------:|------:|--:|
| Critical | 12 | 10 | -2 |
| High | 80 | 61 | -19 |
| Medium | 53 | 57 | +4 |
| Low | 10 | 29 | +19 |
| **Total** | **155** | **157** | **+2** |

The comparison total includes only Critical, High, Medium, and Low findings because the Trivy scan was run with `--severity LOW,MEDIUM,HIGH,CRITICAL`. 
Grype reported 165 findings in total when its additional Negligible and Unknown findings were included.

### Why the difference?
Pick **two specific CVEs** that ONE tool found and the other didn't. For each:
1. CVE ID + tool that found it + tool that missed it
2. Why (likely): different CVE database refresh cadence? Different package matching rules? Different fix-version awareness?

(Lecture 4 mentioned that Grype and Trivy use slightly different DBs; this is where you see it.)

Two examples of tool divergence from this scan:

1. **CVE-2026-48617** was found by Grype in the `node` package (`24.15.0`) but was not reported by Trivy. This may be caused by different database refresh timing or different package and vulnerability matching rules.
2. **CVE-2015-9235** was found by Trivy in the `jsonwebtoken` package (`0.1.0` and `0.4.0`) but was not reported by Grype. This is also consistent with differences in vulnerability databases, ecosystem matching, or version and fix metadata.

### When would you pick each?
2-3 sentences each:
- When does Syft+Grype's **decoupled** model win? (hint: SBOM-as-an-attestation, Lecture 4 + Lab 8)
- When does Trivy's **all-in-one** win? (hint: simpler CI step, broader scope including IaC + secrets + misconfig)

Syft + Grype's decoupled model wins when the SBOM must be stored, reviewed, attested, and rescanned later without rebuilding the image. It is useful for creating a durable software inventory and for Lab 8's signed SBOM workflow.

Trivy's all-in-one model wins when a team wants a simpler CI command that scans an image and can also cover misconfigurations, IaC, and secrets. It reduces pipeline setup when a separate SBOM artifact is not required.

## Task 2: GitHub Actions SBOM + SCA Pipeline

### Workflow file

The workflow is stored at `.github/workflows/lab4-sbom-sca.yml`:

```yaml
name: Lab 4 - SBOM and SCA

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read

env:
  IMAGE: bkimminich/juice-shop:v20.0.0
  REPORT_DIR: labs/lab4/reports

jobs:
  sbom-and-sca:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Create report directory
        run: mkdir -p "${REPORT_DIR}"

      - name: Pull container image
        run: docker pull "${IMAGE}"

      - name: Generate CycloneDX SBOM with Syft
        uses: anchore/sbom-action@v0
        with:
          image: ${{ env.IMAGE }}
          format: cyclonedx-json
          output-file: labs/lab4/reports/juice-shop.cdx.json
          upload-artifact: false

      - name: Generate SPDX SBOM with Syft
        uses: anchore/sbom-action@v0
        with:
          image: ${{ env.IMAGE }}
          format: spdx-json
          output-file: labs/lab4/reports/juice-shop.spdx.json
          upload-artifact: false

      - name: Scan SBOM with Grype
        uses: anchore/scan-action@v7
        with:
          sbom: labs/lab4/reports/juice-shop.cdx.json
          output-format: json
          output-file: labs/lab4/reports/grype-report.json
          fail-build: false

      - name: Scan image with Trivy
        uses: aquasecurity/trivy-action@v0.36.0
        with:
          scan-type: image
          image-ref: ${{ env.IMAGE }}
          scanners: vuln
          severity: LOW,MEDIUM,HIGH,CRITICAL
          format: json
          output: labs/lab4/reports/trivy-report.json
          exit-code: "0"

      - name: Upload security reports
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: lab4-sbom-sca-reports
          path: labs/lab4/reports/
          retention-days: 30
```

### Successful workflow run

- Direct link to a green successful workflow run: `https://github.com/nptkenny12/DevSecOps/actions/runs/32986073975`
- Workflow name: **Lab 4 - SBOM and SCA**
- Job name: **sbom-and-sca**

### Job step explanation

#### Triggers (`on:`)

The workflow runs on pushes to `main`, pull requests targeting `main`, and manual `workflow_dispatch` requests. Running it on both pushes and pull requests provides an additional check before merging and confirms the state of the protected branch after changes arrive.

#### Job: `sbom-and-sca` / `runs-on: ubuntu-latest`

This job generates SBOMs and runs Grype and Trivy scans. A GitHub-hosted Ubuntu runner provides a clean, reproducible environment with Docker support, independent of the developer's local operating system.

#### Step: Pull container image

The workflow pulls `bkimminich/juice-shop:v20.0.0` explicitly so the image is available locally to the Syft and Trivy steps. Pinning the image tag also makes the scan target clear and reproducible.

#### Step: Generate CycloneDX SBOM with Syft

`anchore/sbom-action@v0` generates a CycloneDX inventory from the container image. The output file is used as the input to Grype and can also be retained as a software inventory for later attestation or incident response.

#### Step: Scan SBOM with Grype

`anchore/scan-action@v7` scans the generated CycloneDX SBOM and writes a JSON vulnerability report. `fail-build: false` allows the pipeline to complete and upload reports for analysis even when vulnerabilities are found.

#### Step: Scan image with Trivy

This step scans the container image directly with Trivy instead of scanning the previously generated SBOM. Running both tools provides a comparison between the decoupled Syft-plus-Grype approach and Trivy's all-in-one image scanning approach.

#### Step: Upload security reports

The workflow uploads the generated SBOM and scan reports as the `lab4-sbom-sca-reports` artifact for 30 days. `if: always()` ensures that reports are uploaded even if an earlier step fails.

### Reflection

This CI pipeline complements the local Syft and Grype workflow by providing a centralized scan on GitHub, even when a developer has not installed the local tools or has bypassed a local check. It also regenerates reports in a clean runner and compares Grype with Trivy, giving the team repeatable evidence before and after code is merged.

## Bonus: Sign-Ready SBOM for Lab 8

### CycloneDX schema version

- `specVersion`: `1.5`
- `bomFormat`: `CycloneDX`
- `metadata.timestamp`: `2026-08-26T22:48:28+07:00`
- Syft version: `1.51.0`

### Image digest captured

- `docker inspect bkimminich/juice-shop:v20.0.0 --format '{{index .RepoDigests 0}}'`: `sha256:fd58bdc9745416afce8184ee0666278a436574633ea7880365153a63bfd418b0`

The digest must be captured from the exact `bkimminich/juice-shop:v20.0.0` image and must match the `subject.digest.sha256` value in the attestation.

### Attestation predicate

The attestation must use the in-toto Statement v1 structure below:

```json
{
  "_type": "https://in-toto.io/Statement/v1",
  "subject": [
    {
      "name": "bkimminich/juice-shop:v20.0.0",
      "digest": {
        "sha256": "fd58bdc9745416afce8184ee0666278a436574633ea7880365153a63bfd418b0"
      }
    }
  ],
  "predicateType": "https://cyclonedx.org/bom/v1.5",
  "predicate": {
    "bomFormat": "CycloneDX",
    "specVersion": "1.5",
    "metadata": "...",
    "components": "..."
  }
}
```

The full predicate is stored in `labs/lab4/juice-shop-attestation.json`; the excerpt above is shortened for readability. The complete attestation was validated with `jq`, and its predicate matches `labs/lab4/juice-shop.cdx.json`.

### What this enables in Lab 8

When Lab 8 runs `cosign attest --type cyclonedx --predicate juice-shop-attestation.json`, Cosign will sign a statement containing the image identity and its CycloneDX SBOM. The signed attestation allows a verifier to confirm that the SBOM belongs to the specified Juice Shop image digest and has not been modified after signing.
