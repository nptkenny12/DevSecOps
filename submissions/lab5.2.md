# Lab 5.2 — Submission

## Task 1: DAST with OWASP ZAP

### Baseline unauthenticated scan

- Report: `labs/lab5/results/baseline-report.json`
- Total alert types: **10**
- Duration: Not recorded in the terminal output.

| Severity | Count |
|----------|------:|
| High | 0 |
| Medium | 2 |
| Low | 5 |
| Informational | 3 |
| **Total** | **10** |

### Authenticated full scan

- Report: `labs/lab5/results/auth-report.json`
- Total alert types: **8**
- Duration: Not recorded in the terminal output.

| Severity | Count |
|----------|------:|
| High | 1 |
| Medium | 2 |
| Low | 1 |
| Informational | 4 |
| **Total** | **8** |

### Authentication gap analysis

The authenticated-to-unauthenticated alert ratio was **0.8x** (`8 / 10`). 
This run did not support the expected 10–20x increase: the baseline scan reported more alert types because it discovered several public static-resource and header findings that were not repeated in the authenticated report. 
The authenticated scan nevertheless reached protected functionality and found a high-severity SQL Injection alert on the login endpoint.

### Authenticated-only alert 1

- **Alert:** SQL Injection
- **Severity:** High
- **URL:** `http://juice-shop:3000/rest/user/login`
- **Method and parameter:** `POST`, parameter `email`
- **Payload:** `'`
- **Evidence:** `HTTP/1.1 500 Internal Server Error`
- **Reason:** The authenticated Automation Framework scan actively tested the login API with an injection payload. The baseline scan did not produce this active SQL Injection alert for the login request.

### Authenticated-only alert 2

- **Alert:** Session Management Response Identified
- **Severity:** Informational
- **URL:** `http://juice-shop:3000/rest/user/login`
- **Parameter:** `authentication.token`
- **Reason:** The authenticated scan submitted the login request and observed the session token in the response. This session-management evidence was not present in the baseline report.

### Commands executed

```bash
docker run --rm \
  --network lab5-net \
  --user root \
  -v "$(pwd)/labs/lab5/results:/zap/wrk" \
  ghcr.io/zaproxy/zaproxy:stable \
  zap-baseline.py \
  -t http://juice-shop:3000 \
  -r baseline-report.html \
  -J baseline-report.json
```

```bash
docker run --rm \
  --network lab5-net \
  --user root \
  -e _JAVA_OPTIONS="-Xmx1g" \
  -v "$(pwd)/labs/lab5:/zap/wrk" \
  ghcr.io/zaproxy/zaproxy:stable \
  zap.sh -cmd \
  -autorun /zap/wrk/scripts/zap-auth.yaml \
  -port 8090
```

```bash
bash labs/lab5/scripts/compare_zap.sh \
  labs/lab5/results/baseline-report.json \
  labs/lab5/results/auth-report.json
```

### Observable output

```text
Unauthenticated Scan:
  Total alerts: 10
  High: 0
  Medium: 2
  Low: 5
  Info: 3

Authenticated Scan:
  Total alerts: 8
  High: 1
  Medium: 2
  Low: 1
  Info: 4
```

The baseline scan returned exit code `2`, which means ZAP found issues; both JSON reports were created and validated successfully.

## Bonus: SAST/DAST Correlation

### Correlation table

Semgrep completed successfully after correcting the scanning root and produced **27 findings**. Parser warnings were reported for intentionally malformed challenge fixtures and some Angular template files, but the JSON report was generated successfully.

| # | OWASP category | ZAP alert | ZAP URI | Semgrep rule | Semgrep file:line | Confidence |
|---|----------------|-----------|----------|--------------|------------------|------------|
| 1 | A03:2025 Injection | SQL Injection (High) | `/rest/user/login` | `javascript.sequelize.security.audit.sequelize-injection-express.express-sequelize-injection` | `routes/login.ts:34` | High — both tools agree |

### Strongest correlation deep-dive

**Vulnerable code:**

```ts
models.sequelize.query(`SELECT * FROM Users WHERE email = '${req.body.email || ''}' AND password = '${security.hash(req.body.password || '')}' AND deletedAt IS NULL`, { model: UserModel, plain: true })
```

**Working payload and DAST evidence:**

- Request: `POST http://juice-shop:3000/rest/user/login`
- Parameter: `email`
- Payload: `'`
- Evidence: `HTTP/1.1 500 Internal Server Error`

**Proposed fix:** Replace string interpolation with a parameterized Sequelize query or a safe ORM query method. User-controlled values such as `email` and `password` must be passed as bind/replacement parameters, never concatenated into SQL text.

**Why both tools caught it:** Semgrep detected the unsafe data flow statically because `req.body.email` is interpolated into a SQL query. ZAP confirmed the issue dynamically by sending the quote payload to the login endpoint and observing the resulting server error. This agreement makes the finding high confidence.

### Reflection

Lecture 5 slide 15 describes agreement between SAST and DAST as the highest-confidence finding type. 
In a real PR review, I would want the SAST finding first because it identifies the vulnerable source line early and helps prevent the issue before deployment; DAST evidence then confirms that the running application is exploitable.

### Semgrep command and observable output

```bash
semgrep \
  --config=p/owasp-top-ten \
  --config=p/javascript \
  --config=p/secrets \
  labs/lab5/semgrep/juice-shop \
  --json -o labs/lab5/results/semgrep.json \
  --severity ERROR --severity WARNING
```

```text
Semgrep findings: 27
```
