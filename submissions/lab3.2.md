# Lab 3.2 Submission - Secure Git: History Hygiene

## Task 2 Continued: Gitleaks Tune-out

### Inline allowlist

The inline `# gitleaks:allow` comment suppresses a finding only on the specific intentional test line. Other files continue to be scanned normally.

Example:

```text
EXAMPLE_TOKEN=not-a-real-secret  # gitleaks:allow
```

### Path exclusion

The `.gitleaks.toml` file excludes only files under `docs/examples/`.

Command executed:

```bash
pre-commit run gitleaks --all-files
```
Observed output:
```text
Detect hardcoded secrets.................................................Passed
```

## Bonus: History Rewrite

### Before

The sandbox history before rewriting contained these commits:

```text
a28b69d (HEAD -> main) docs: add usage notes
be44cf1 feat: empty log
7540e47 feat: add config
df74c09 init
```

Output of `git log -p | grep -c 'ghp_'`: **2**

### After

The rewritten sandbox history was:

```text
de14b7f (HEAD -> main) docs: add usage notes
7d537b7 feat: empty log
2e627ca feat: add config
ed8761b init
```

The rewrite was performed with:

```bash
git filter-repo --replace-text /tmp/replace.txt --force
```

Output of `git log -p | grep -c 'ghp_'`: **0**

Output of `git log -p | grep -c 'REDACTED'`: **2**

### The two-step pattern in real life

1. Rewrite the Git history: `git filter-repo --replace-text /tmp/replace.txt`
2. Rotate or revoke the exposed secret.
Rewriting history removes the secret from reachable Git commits, but it does not invalidate a credential that may already have been copied.

### Two real-world gotchas you discovered

1. `git-filter-repo` refused to run because the sandbox was not detected as a fresh clone; I had to use `--force` because this repository was disposable.
2. History rewriting changes commit IDs because every affected commit receives a new hash, so collaborators must re-clone or carefully reset their local branches.
