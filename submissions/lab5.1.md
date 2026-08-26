# Lab 5.1 — Submission

## Task 1: SAST with Semgrep

### Scan target and commands

The Juice Shop source was cloned at tag `v20.0.0` so that the scan matches the version used in the lab container.

```bash
git -C labs/lab5/semgrep/juice-shop describe --tags --always
```

Observed output:

```text
v20.0.0
```

```bash
semgrep \
  --config=p/owasp-top-ten \
  --config=p/javascript \
  --config=p/secrets \
  labs/lab5/semgrep/juice-shop \
  --json -o labs/lab5/results/semgrep.json \
  --severity ERROR --severity WARNING
```

### Semgrep severity breakdown

| Severity | Count |
|----------|------:|
| ERROR | 13 |
| WARNING | 14 |
| INFO | 0 |
| **Total** | **27** |

### Top rules by frequency

| Rule ID | Count | OWASP category |
|---------|------:|----------------|
| `javascript.sequelize.security.audit.sequelize-injection-express.express-sequelize-injection` | 6 | A05:2025 Injection |
| `yaml.github-actions.security.run-shell-injection.run-shell-injection` | 5 | A05:2025 Injection |
| `javascript.express.security.audit.express-check-directory-listing.express-check-directory-listing` | 4 | A01:2025 Broken Access Control |
| `javascript.express.security.audit.express-res-sendfile.express-res-sendfile` | 4 | A06:2025 Insecure Design |
| `yaml.github-actions.security.github-actions-mutable-action-tag.github-actions-mutable-action-tag` | 4 | A08:2025 Software and Data Integrity Failures |
| `javascript.express.security.audit.express-open-redirect.express-open-redirect` | 1 | A01:2025 Broken Access Control |
| `javascript.jsonwebtoken.security.jwt-hardcode.hardcoded-jwt-secret` | 1 | A07:2025 Authentication Failures |
| `javascript.lang.security.audit.code-string-concat.code-string-concat` | 1 | A05:2025 Injection |
| `yaml.github-actions.security.gha-curl-pipe-shell.gha-curl-pipe-shell` | 1 | A03:2025 Injection |

### Triage shortcut (Lecture 5 slide 8)

I would fix `javascript.sequelize.security.audit.sequelize-injection-express.express-sequelize-injection` first because it is the most frequent rule, with six findings, and it represents a high-impact SQL injection risk. 
Fixing the shared query pattern and replacing string concatenation with parameterized queries could remove multiple findings at the module level.

### False-positive sample

- **File:** `labs/lab5/semgrep/juice-shop/data/static/codefixes/dbSchemaChallenge_1.ts:5`
- **Rule:** `javascript.sequelize.security.audit.sequelize-injection-express.express-sequelize-injection`
- **Reason:** This file is an intentionally stored vulnerable code sample used by a Juice Shop challenge and is not part of the normal application execution path.
I would suppress this specific fixture finding only after confirming that the directory is excluded from production builds and that the sample is required for the educational challenge.

The finding should not be suppressed in normal application code, where the same unparameterized query pattern could allow SQL injection.
