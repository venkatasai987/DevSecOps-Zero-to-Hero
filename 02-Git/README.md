# DevSecOps for Git

## Contents

1. .gitignore  
2. Native Git Pre-Commit Hooks (Custom Scripts)  
3. Gitleaks — Blocking Commits  
4. Gitleaks — Repository & History Scanning  
5. Gitleaks Installation Script  
6. Gitleaks in GitHub Actions  
7. Branch Protection Rules  
8. RBAC  
9. Mandatory Reviews  
10. Signed Commits  
11. CODEOWNERS  
12. Dependabot  

---

## 1️⃣ .gitignore — First Line of Defense

### Purpose
Prevent sensitive files from ever being tracked by Git.

### Common Security Files to Ignore
```gitignore
.env
.env.*
*.pem
*.key
id_rsa
terraform.tfstate
.terraform/
node_modules/
dist/
```

### Demo
```bash
echo "AWS_SECRET_ACCESS_KEY=123" > .env
git status
```

Add `.gitignore`:
```bash
echo ".env" >> .gitignore
git status
```

✅ File is no longer tracked.

⚠️ `.gitignore` does NOT protect secrets already committed.

---

## 2️⃣ Native Git Pre-Commit Hooks (Custom Script)

### What This Is
A **pre-commit hook** is a script located at:

```text
.git/hooks/pre-commit
```

Git executes it **automatically before every commit**.

### Exit Codes
| Code | Result |
|------|--------|
| 0    | Commit allowed |
| ≠0   | Commit blocked |

---

### Demo — Minimal Native Secret Detector

```bash
cat > .git/hooks/pre-commit << 'EOF'
#!/bin/bash

echo "🔍 Running native pre-commit hook..."

if git diff --cached | grep -i "secret"; then
  echo "❌ Secret detected. Commit blocked."
  exit 1
fi

echo "✅ Commit passed security checks."
exit 0
EOF
```

Make executable:
```bash
chmod +x .git/hooks/pre-commit
```

Test:
```bash
echo "my_secret=123" > test.txt
git add test.txt
git commit -m "test commit"
```

❌ Commit blocked.

---

## 3️⃣ Gitleaks — Blocking Commits (Native Hook)

### Purpose
Use **real secret detection** instead of simple pattern matching.

---

### Replace Pre-Commit Hook with Gitleaks

```bash
cat > .git/hooks/pre-commit << 'EOF'
#!/bin/bash

echo "🔍 Running Gitleaks pre-commit scan..."

gitleaks protect --staged
STATUS=$?

if [ $STATUS -ne 0 ]; then
  echo "❌ Gitleaks detected secrets. Commit blocked."
  exit 1
fi

echo "✅ No secrets detected."
exit 0
EOF
```

Make executable:
```bash
chmod +x .git/hooks/pre-commit
```

---

### Demo — Block a Secret Commit

```bash
echo "AWS_SECRET_ACCESS_KEY=AKIA123456789" > secrets.env
git add secrets.env
git commit -m "adding secrets"
```

❌ Commit blocked.

---

## 4️⃣ Gitleaks — Repository & History Scanning

### Scan Working Tree
```bash
gitleaks detect --source .
```

---

### Scan Full Git History
```bash
gitleaks detect --source . --log-opts="--all"
```
---

## 5️⃣ Gitleaks Installation Script

```bash
#!/bin/bash
set -e

VERSION=8.18.1
curl -sSL https://github.com/gitleaks/gitleaks/releases/download/v$VERSION/gitleaks-linux-amd64 \
  -o /usr/local/bin/gitleaks

chmod +x /usr/local/bin/gitleaks
gitleaks version
```

---

## 6️⃣ Gitleaks in GitHub Actions

```yaml
name: Secret Scan

on: [pull_request]

jobs:
  gitleaks:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: gitleaks/gitleaks-action@v2
```

---

## 7️⃣ Branch Protection Rules

Enforce:
- No direct pushes to `main`
- Required pull requests
- Required status checks
- No force pushes

---

## 8️⃣ RBAC — Least Privilege

| Role | Permissions |
|-----|-------------|
| Admin | Repo settings |
| Maintainer | Merge PRs |
| Developer | PR only |
| Auditor | Read-only |

---

## 9️⃣ Mandatory Reviews

Best practices:
- Minimum 1–2 reviewers
- Code owners for sensitive paths
- Security review for auth, infra, CI

---

## 🔟 Signed Commits

```bash
git commit -S -m "secure commit"
git log --show-signature
```

Verified commits show a **Verified** badge.

---

## 1️⃣1️⃣ CODEOWNERS

```text
/.github/ @security-team
/terraform/ @cloud-team
```

---

## 1️⃣2️⃣ Dependabot

```yaml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
```

---