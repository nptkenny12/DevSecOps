# Lab 4.1 — Submission

## Task 1: Syft + Grype on Juice Shop

### SBOM stats

- `juice-shop.cdx.json` component count: `3068`
- `juice-shop.cdx.json` size: `1.7M `
- `juice-shop.spdx.json` package count: `909`

### Grype severity breakdown

| Severity | Count |
|----------|------:|
| Critical | 12 |
| High | 80 |
| Medium | 53 |
| Low | 10 |
| Negligible | 7 |
| Unknown | 3 |
| **Total** | **165** |

### Top 10 CVEs

| CVE | Severity | Package | Installed | Fix |
|-----|----------|---------|-----------|-----|
| CVE-2026-34182 | Critical | libssl3t64 | 3.5.5-1~deb13u2 | 3.5.6-1~deb13u2 |
| CVE-2026-5450 | Critical | libc6 | 2.41-12+deb13u2 | No fix available |
| GHSA-23hp-3jrh-7fpw | Critical | tar | 4.4.19 | 7.5.19 |
| GHSA-23hp-3jrh-7fpw | Critical | tar | 6.2.1 | 7.5.19 |
| GHSA-23hp-3jrh-7fpw | Critical | tar | 7.5.15 | 7.5.19 |
| GHSA-5mrr-rgp6-x4gr | Critical | marsdb | 0.6.11 | No fix available |
| GHSA-c7hr-j4mj-j2w6 | Critical | jsonwebtoken | 0.1.0 | 4.2.2 |
| GHSA-c7hr-j4mj-j2w6 | Critical | jsonwebtoken | 0.4.0 | 4.2.2 |
| GHSA-jf85-cpcp-j695 | Critical | lodash | 2.4.2 | 4.17.12 |
| GHSA-mp2f-45pm-3cg9 | Critical | decompress | 4.2.1 | No fix available |

### Fix-available analysis

Out of the top 10 findings, **8 have a fix available** and 2 do not currently list a fix. 
The Critical findings with an available fix should receive the highest patching priority because remediation is immediately actionable. 
Following Lecture 4's triage shortcut, vulnerabilities with severity High or Critical and an available fix should be addressed first.
