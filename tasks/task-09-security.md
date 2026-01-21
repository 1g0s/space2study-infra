# Task 9: Security & Vulnerability Scanning - Space2Study

**Status:** In Progress
**Started:** 2026-01-21
**Completed:** -

---

## Objective

Implement comprehensive security scanning across all repositories to detect vulnerabilities in dependencies, code, containers, and infrastructure.

---

## Security Options Available

| Feature | Type | Cost | Description |
|---------|------|------|-------------|
| **Dependabot Alerts** | Dependency Scanning | Free | Scans package.json for known CVEs |
| **Dependabot Updates** | Auto-fix | Free | Creates PRs to update vulnerable dependencies |
| **Secret Scanning** | Credential Detection | Free | Detects leaked API keys, tokens, passwords |
| **CodeQL** | SAST (Static Analysis) | Free | Finds SQL injection, XSS, security bugs in code |
| **Trivy** | Container Scanning | Free | Scans Docker images for vulnerabilities |
| **Trivy IaC** | Infrastructure Scanning | Free | Scans Terraform for misconfigurations |
| **Snyk** | Multi-purpose | Free tier | Dependencies, containers, code, IaC |
| **Grype** | Container Scanning | Free | Alternative to Trivy for containers |

---

## Recommended Setup

### 1. Dependabot Configuration

Create in each repository:

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 5

  - package-ecosystem: "docker"
    directory: "/"
    schedule:
      interval: "weekly"

  - package-ecosystem: "terraform"
    directory: "/terraform"
    schedule:
      interval: "weekly"
```

### 2. Security Scanning Workflow (Trivy)

```yaml
# .github/workflows/security.yml
name: Security Scan

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 6 * * 1'  # Weekly Monday 6am

jobs:
  trivy-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run Trivy vulnerability scanner (repo)
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          scan-ref: '.'
          severity: 'CRITICAL,HIGH'
          format: 'sarif'
          output: 'trivy-results.sarif'

      - name: Upload Trivy scan results to GitHub Security
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: 'trivy-results.sarif'

  trivy-iac:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run Trivy IaC scanner (Terraform)
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'config'
          scan-ref: './terraform'
          severity: 'CRITICAL,HIGH,MEDIUM'
          format: 'table'
```

### 3. CodeQL for SAST (JavaScript/TypeScript)

```yaml
# .github/workflows/codeql.yml
name: CodeQL Analysis

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 6 * * 1'

jobs:
  analyze:
    runs-on: ubuntu-latest
    permissions:
      security-events: write

    strategy:
      matrix:
        language: ['javascript']

    steps:
      - uses: actions/checkout@v4

      - name: Initialize CodeQL
        uses: github/codeql-action/init@v3
        with:
          languages: ${{ matrix.language }}

      - name: Autobuild
        uses: github/codeql-action/autobuild@v3

      - name: Perform CodeQL Analysis
        uses: github/codeql-action/analyze@v3
```

---

## Security Scanning Layers

```mermaid
flowchart TB
    subgraph "GitHub Repository"
        Code[Source Code]
        Deps[Dependencies]
        Docker[Dockerfiles]
        TF[Terraform]
    end

    subgraph "Security Tools"
        CodeQL[CodeQL SAST]
        Dependabot[Dependabot]
        Trivy[Trivy Scanner]
        Secrets[Secret Scanning]
    end

    subgraph "Results"
        Alerts[Security Alerts]
        PRs[Auto-fix PRs]
        SARIF[SARIF Reports]
    end

    Code --> CodeQL --> Alerts
    Code --> Secrets --> Alerts
    Deps --> Dependabot --> PRs
    Docker --> Trivy --> SARIF
    TF --> Trivy --> SARIF
```

---

## Repositories to Configure

| Repository | Dependabot | CodeQL | Trivy | Secret Scanning |
|------------|------------|--------|-------|-----------------|
| space2study-backend | npm, docker | javascript | fs, container | Yes |
| space2study-frontend | npm, docker | javascript | fs, container | Yes |
| space2study-infra | terraform, docker | - | config (IaC) | Yes |

---

## Implementation Steps

### Phase 1: Enable GitHub Built-in Features

1. **Enable Dependabot Alerts**
   - Go to each repo → Settings → Security → Code security and analysis
   - Enable "Dependabot alerts"
   - Enable "Dependabot security updates"

2. **Enable Secret Scanning**
   - Same location → Enable "Secret scanning"
   - Enable "Push protection" (blocks commits with secrets)

### Phase 2: Add Dependabot Configuration

1. Create `.github/dependabot.yml` in each repo
2. Configure for npm (backend, frontend) and terraform (infra)
3. Commit and push

### Phase 3: Add Security Workflows

1. **Backend repo:**
   - Add `.github/workflows/security.yml` (Trivy)
   - Add `.github/workflows/codeql.yml` (JavaScript)

2. **Frontend repo:**
   - Add `.github/workflows/security.yml` (Trivy)
   - Add `.github/workflows/codeql.yml` (JavaScript)

3. **Infra repo:**
   - Add `.github/workflows/security.yml` (Trivy IaC only)

### Phase 4: Review and Fix Findings

1. Check Security tab in each repo for alerts
2. Review Dependabot PRs
3. Fix CRITICAL and HIGH severity issues
4. Document accepted risks for false positives

---

## Deliverables Checklist

### GitHub Settings
- [ ] Enable Dependabot alerts (backend)
- [ ] Enable Dependabot alerts (frontend)
- [ ] Enable Dependabot alerts (infra)
- [ ] Enable Secret Scanning (backend)
- [ ] Enable Secret Scanning (frontend)
- [ ] Enable Secret Scanning (infra)

### Dependabot Configuration
- [x] Add dependabot.yml (backend)
- [x] Add dependabot.yml (frontend)
- [x] Add dependabot.yml (infra)

### Security Workflows
- [x] Add security.yml - Trivy (backend)
- [x] Add security.yml - Trivy (frontend)
- [x] Add security.yml - Trivy IaC (infra)
- [x] Add codeql.yml (backend)
- [x] Add codeql.yml (frontend)

### Verification
- [ ] All workflows passing
- [ ] Review initial security findings
- [ ] Fix CRITICAL vulnerabilities
- [ ] Document accepted risks

---

## Verification Commands

```bash
# Check GitHub Security tab
# https://github.com/DevOps-ProjectLevel/space2study-backend-1g0s/security
# https://github.com/DevOps-ProjectLevel/space2study-frontend-1g0s/security
# https://github.com/1g0s/space2study-infra/security

# View Dependabot alerts via CLI
gh api repos/DevOps-ProjectLevel/space2study-backend-1g0s/dependabot/alerts

# View code scanning alerts
gh api repos/DevOps-ProjectLevel/space2study-backend-1g0s/code-scanning/alerts
```

---

## Expected Outcomes

After implementation:
- Automatic weekly dependency vulnerability scans
- PRs created automatically for vulnerable dependencies
- Static code analysis on every push/PR
- Container image scanning before deployment
- Terraform misconfiguration detection
- Secret leak prevention with push protection
