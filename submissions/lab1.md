# Lab 1  Submission

## Triage Report: OWASP Juice Shop

### Scope & Asset
- Asset: OWASP Juice Shop (local lab instance)
- Image: `bkimminich/juice-shop:v20.0.0`
- Image digest: `sha256:fd58bdc9745416afce8184ee0666278a436574633ea7880365153a63bfd418b0`
- Host OS: `macOS 27.0, arm64`
- Docker version: `Docker version 29.7.2, build a7dcaa6`
---
### Deployment Details
- Run command used: `docker run -d --name juice-shop -p 127.0.0.1:3000:3000 bkimminich/juice-shop:v20.0.0`
- Access URL: `http://127.0.0.1:3000`
- Network exposure: 127.0.0.1 only? ` [ ] No (explain if No)
- Container restart policy: `no`
<!-- The application is bound only to localhost. -->
---
### Health Check
- HTTP code on `/`: `200`
- API check (first 200 chars of `/api/Products`): 
```json
{"status":"success","data":[{"id":1,"name":"Apple Juice (1000ml)","description":"The all-time classic.","price":1.99,"deluxePrice":0.99,"image":"apple_juice.jpg","createdAt":"2026-08-19T09:46:38.765Z"
```
- Container uptime: 
```text
CONTAINER ID   IMAGE                           COMMAND                  CREATED      STATUS          PORTS                      NAMES
31cc245235b3   bkimminich/juice-shop:v20.0.0   "/nodejs/bin/node /j…"   4 days ago   Up 35 minutes   127.0.0.1:3000->3000/tcp   juice-shop
```
### Initial Surface Snapshot (from browser exploration)
- Login/Registration visible: [x] `Yes` [ ] No . Notes: `Login page, User Registration form.`
- Product listing/search present: [x] `Yes` [ ] No . Notes: `The homepage displayed the All Products listing, search control, pagination, and 46 products.`
- Admin or account area discoverable: [x] `Yes` [ ] No . Notes: `The Account menu exposed the Login page. No Admin link was visible while unauthenticated.`
- Client-side errors in DevTools console: [x] `Yes` [ ] No . Notes: `No client-side errors or warnings were observed`
- Pre-populated local storage / cookies:
  - Cookie: `language=en`
  - Cookie: `welcomebanner...=dismiss`
  - Local Storage: no entries observed

### Security Headers (Quick Look)
Run: `curl -I http://127.0.0.1:3000 2>&1 | head -20`. Paste output:
```text
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
Feature-Policy: payment 'self'
X-Recruiting: /#/jobs
Accept-Ranges: bytes
Cache-Control: public, max-age=0
Last-Modified: Sun, 23 Aug 2026 12:27:56 GMT
ETag: W/"26af-1a02e9781ec"
Content-Type: text/html; charset=UTF-8
Content-Length: 9903
Vary: Accept-Encoding
Date: Sun, 23 Aug 2026 13:31:50 GMT
Connection: keep-alive
Keep-Alive: timeout=5
```
Which of these are MISSING? (cross-reference Lecture 1 OWASP Top 10:2025  A06)
- [x] `Content-Security-Policy`
- [x] `Strict-Transport-Security`
- [ ] X-Content-Type-Options: nosniff
- [ ] X-Frame-Options

### Top 3 Risks Observed
1. `Missing security headers` — No CSP or HSTS headers.  
   **OWASP A06: Security Misconfiguration**
2. `Unauthenticated review data exposure` — Reviewer email addresses are publicly accessible.  
   **OWASP A01: Broken Access Control**
3. `Stack-trace disclosure` — Error responses reveal internal paths and framework details.  
   **OWASP A10: Mishandling of Exceptional Conditions**

## PR Template Setup
- File: `.github/PULL_REQUEST_TEMPLATE.md`
- Sections included: Goal / Changes / Testing / Artifacts & Screenshots
- Checklist items:
  - PR title follows `feat(labN): <topic>`
  - No secrets or large temporary files are committed
  - `submissions/lab1.md` exists
- Auto-fill verified: [ ] Yes
- Draft PR evidence: Pending - create a draft PR and add its URL here.

## Lab Completion Checklist
- [x] Task 1 — Juice Shop deployed and triage report completed
- [x] Task 2 — PR template created and auto-fill verified
- [ ] Bonus — CI smoke test runs successfully