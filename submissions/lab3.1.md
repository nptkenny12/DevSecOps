# Lab 3.1  Submission

## Task 1: SSH Commit Signing

### Local configuration
- git config --global gpg.format : `ssh`
- git config --global user.signingkey  : `/Users/tien.p.nguyen/.ssh/id_ed25519.pub`
- git config --global commit.gpgsign: `true`

### Local verification
- Output of `git log --show-signature -1`:
```text
commit d6361bcc690d8d1a77fffab9cac7c73c24de3f95 (HEAD -> feature/lab3.1, origin/feature/lab3.1)
Good "git" signature for nptkenny@gmail.com with ED25519 key [fingerprint redacted]
Author: Kenny <nptkenny@gmail.com>
Date: Mon Aug 24 21:08:53 2026 +0700

test: first signed commit
```
### GitHub verification
- Direct link to your most recent commit on GitHub: https://github.com/nptkenny12/DevSecOps/commit/d6361bcc690d8d1a77fffab9cac7c73c24de3f95
- Screenshot of the Verified badge:

![Verified badge](screenshots/verified_badge.png)

### One-paragraph reflection (2-3 sentences)
A forged-author commit could make it appear that a trusted developer introduced a malicious change, allowing the attacker to deny responsibility and complicating incident investigation. 
The Verified badge makes the attack visible by confirming whether the commit was signed with a key linked to the claimed GitHub identity; an unsigned or unverified commit would raise suspicion.

## ## Task 2: Pre-commit + gitleaks

### `.pre-commit-config.yaml`

```yaml
repos:
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.30.1
    hooks:
      - id: gitleaks

  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v5.0.0
    hooks:
      - id: detect-private-key
      - id: check-added-large-files
```

### `pre-commit install` output

```text
pre-commit installed at .git/hooks/pre-commit
```

### Blocked commit

```text
[WARNING] Unstaged files detected.
[INFO] Stashing unstaged files to /Users/tien.p.nguyen/.cache/pre-commit/patch1787583140-21914.
Detect hardcoded secrets.................................................Failed
- hook id: gitleaks
- exit code: 1

○
    │╲
    │ ○
    ○ ░
    ░    gitleaks

Finding:     GH_PAT=REDACTED
Secret:      REDACTED
RuleID:      github-pat
Entropy:     4.143943
File:        submissions/leak-attempt.txt
Line:        2
Fingerprint: submissions/leak-attempt.txt:github-pat:2

9:52PM INF 0 commits scanned.
9:52PM INF scanned ~91 bytes (91 bytes) in 17.9ms
9:52PM WRN leaks found: 1

detect private key.......................................................Passed
check for added large files..............................................Passed


```