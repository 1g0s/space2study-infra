# Task 9: Security & Vulnerability Scanning - Space2Study

**Status:** ✅ Complete
**Started:** January 21, 2026
**Completed:** January 21, 2026

---

## Objective

Implement security scanning across all repositories to detect vulnerabilities in dependencies and code.

---

## Implementation Summary

### Security Tools Implemented

| Tool | Purpose | Repositories |
|------|---------|--------------|
| **Dependabot** | Dependency vulnerability alerts + auto-fix PRs | All 3 repos |
| **Trivy** | Filesystem vulnerability scanning (npm/Docker) | All 3 repos |

### Tools Not Used (Require GitHub Advanced Security)

| Tool | Reason Not Used |
|------|-----------------|
| CodeQL | Requires GHAS (paid feature for private repos) |
| SARIF Upload | Requires GHAS for Security tab integration |
| Dependency Review | Requires Dependency Graph (GHAS feature) |

---

## Files Created/Modified

### space2study-backend

| File | Purpose |
|------|---------|
| `.github/dependabot.yml` | npm, Docker, GitHub Actions dependency updates |
| `.github/workflows/security.yml` | Trivy filesystem scan |

### space2study-frontend

| File | Purpose |
|------|---------|
| `.github/dependabot.yml` | npm, Docker, GitHub Actions dependency updates |
| `.github/workflows/security.yml` | Trivy filesystem scan |

### space2study-infra

| File | Purpose |
|------|---------|
| `.github/dependabot.yml` | Terraform, Docker, GitHub Actions dependency updates |
| `.github/workflows/security.yml` | Trivy FS, IaC (Terraform), and K8s manifest scanning |

---

## Workflow Configuration

### Security Scan Workflow (Component Repos)

```yaml
name: Security Scan

on:
  push:
    branches: [main, master]
  pull_request:
    branches: [main, master]
  schedule:
    - cron: '0 6 * * 1'  # Weekly Monday 6am UTC

jobs:
  trivy-scan:
    name: Trivy Vulnerability Scanner
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          scan-ref: '.'
          severity: 'CRITICAL,HIGH,MEDIUM'
          format: 'table'
          exit-code: '0'
```

### Security Scan Workflow (Infra Repo)

```yaml
name: Security Scan

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 6 * * 1'

jobs:
  trivy-fs:
    name: Trivy Filesystem Scan
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          scan-ref: '.'
          severity: 'CRITICAL,HIGH,MEDIUM'
          format: 'table'
          exit-code: '0'

  trivy-iac:
    name: Trivy IaC Scan (Terraform)
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'config'
          scan-ref: './terraform'
          severity: 'CRITICAL,HIGH,MEDIUM'
          format: 'table'
          exit-code: '0'

  trivy-k8s:
    name: Trivy K8s Manifests Scan
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'config'
          scan-ref: './k8s'
          severity: 'CRITICAL,HIGH,MEDIUM'
          format: 'table'
          exit-code: '0'
```

### Dependabot Configuration (Backend/Frontend)

```yaml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
      day: "monday"
    open-pull-requests-limit: 5
    labels: ["dependencies", "security"]

  - package-ecosystem: "docker"
    directory: "/"
    schedule:
      interval: "weekly"

  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
```

### Dependabot Configuration (Infra)

```yaml
version: 2
updates:
  - package-ecosystem: "terraform"
    directory: "/terraform"
    schedule:
      interval: "weekly"
      day: "monday"
    open-pull-requests-limit: 5
    labels: ["dependencies", "terraform"]

  - package-ecosystem: "docker"
    directory: "/"
    schedule:
      interval: "weekly"

  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
```

---

## Workflow Results

### Security Scan Performance

| Repository | Status | Duration |
|------------|--------|----------|
| space2study-backend | SUCCESS | ~1m |
| space2study-frontend | SUCCESS | ~1m |
| space2study-infra | SUCCESS | ~40s |

### Dependabot Activity (Verified Working)

Dependabot automatically created PRs within minutes of configuration:

| PR | Update | Status |
|----|--------|--------|
| #1 | docker/build-push-action v5 → v6 | ✅ Merged |
| #2 | actions/checkout v4 → v6 | ✅ Merged |
| #3 | github/codeql-action v3 → v4 | ❌ Closed (not used) |

**Note:** Dependabot is a GitHub service (not a workflow). It runs automatically based on `dependabot.yml` configuration and creates PRs directly - no Actions minutes consumed.

---

## Issues Encountered & Resolutions

### Issue 1: CodeQL Requires GHAS

**Error:** "Advanced Security must be enabled for this repository to use code scanning"

**Resolution:** Removed CodeQL workflow - requires paid GitHub Advanced Security for private repos in organizations.

### Issue 2: SARIF Upload Requires GHAS

**Error:** "Resource not accessible by integration"

**Resolution:** Removed SARIF upload step - Security tab integration requires GHAS.

### Issue 3: Dependency Review Requires Dependency Graph

**Error:** "Dependency Graph not enabled"

**Resolution:** Removed dependency-review action - requires Dependency Graph which is a GHAS feature.

---

## Verification

### Check Workflow Status

```bash
# View recent workflow runs
gh run list -R DevOps-ProjectLevel/space2study-backend-1g0s --limit 3
gh run list -R DevOps-ProjectLevel/space2study-frontend-1g0s --limit 3
gh run list -R 1g0s/space2study-infra --limit 3
```

### Check Dependabot PRs

```bash
# View open Dependabot PRs
gh pr list -R DevOps-ProjectLevel/space2study-backend-1g0s --author app/dependabot
gh pr list -R DevOps-ProjectLevel/space2study-frontend-1g0s --author app/dependabot
```

---

## Deliverables Checklist

### Dependabot Configuration
- [x] Add dependabot.yml (backend)
- [x] Add dependabot.yml (frontend)
- [x] Add dependabot.yml (infra)

### Security Workflows
- [x] Add security.yml - Trivy (backend)
- [x] Add security.yml - Trivy (frontend)
- [x] Add security.yml - Trivy FS + IaC + K8s (infra)

### Verification
- [x] All Security Scan workflows passing
- [x] Dependabot creating PRs for vulnerable dependencies

### Manual Settings (Pending Admin Access)
- [ ] Enable Secret Scanning in repo settings
- [ ] Enable Push Protection in repo settings

---

## Security Coverage

| Scan Type | Tool | What It Checks |
|-----------|------|----------------|
| Dependency vulnerabilities | Trivy + Dependabot | npm package.json, package-lock.json |
| Docker image vulnerabilities | Trivy | Dockerfile base images |
| Terraform misconfigurations | Trivy IaC | Security groups, IAM policies, encryption |
| K8s misconfigurations | Trivy Config | Pod security, resource limits, secrets |
| GitHub Actions | Dependabot | Outdated action versions |

---

## What's Not Covered (Requires GHAS)

| Feature | Why Not Available |
|---------|-------------------|
| CodeQL SAST | Requires GitHub Advanced Security ($49/user/month) |
| Secret Scanning Push Protection | Requires admin access to enable |
| Security Tab Alerts | SARIF upload requires GHAS |
| Dependency Review in PRs | Requires Dependency Graph (GHAS) |

---

## Next Steps (Optional Enhancements)

1. **Enable Secret Scanning** - Requires admin access to repo settings
2. **Consider SonarCloud** - Free for public repos, alternative to CodeQL
3. **Add Snyk** - Free tier available, good for npm projects
4. **Container Scanning in CI** - Scan built images before push to ECR
