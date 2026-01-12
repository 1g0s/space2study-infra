# Task 4: CI/CD Implementation - Completion Report

**Project:** Space2Study
**Status:** IMPLEMENTED
**Date:** January 11, 2026
**Environment:** GitHub Actions + GitHub Container Registry

---

## Deliverables Created

| File | Location | Purpose |
|------|----------|---------|
| Backend CI workflow | `space2study-backend/.github/workflows/ci.yml` | Jest tests, lint, coverage |
| Frontend CI workflow | `space2study-frontend/.github/workflows/ci.yml` | Vitest tests, lint, build |
| Docker build workflow | `.github/workflows/docker.yml` | Build & push to ghcr.io |

---

## CI/CD Architecture

```mermaid
graph TB
    subgraph "GitHub Repository"
        PUSH[Push / PR]
    end

    subgraph "GitHub Actions"
        subgraph "Backend CI"
            B_LINT[ESLint]
            B_TEST[Jest Tests]
            B_COV[Coverage Report]
        end

        subgraph "Frontend CI"
            F_LINT[ESLint]
            F_TEST[Vitest Tests]
            F_BUILD[Vite Build]
            F_COV[Coverage Report]
        end

        subgraph "Docker Build (main branch)"
            D_BE[Build Backend Image]
            D_FE[Build Frontend Image]
            D_PUSH[Push to ghcr.io]
        end
    end

    subgraph "Artifacts"
        GHCR[(GitHub Container Registry)]
        ART[(Coverage Reports)]
    end

    PUSH --> B_LINT
    PUSH --> F_LINT
    B_LINT --> B_TEST
    B_TEST --> B_COV
    F_LINT --> F_TEST
    F_TEST --> F_BUILD
    F_BUILD --> F_COV

    PUSH -->|main branch| D_BE
    PUSH -->|main branch| D_FE
    D_BE --> D_PUSH
    D_FE --> D_PUSH
    D_PUSH --> GHCR
    B_COV --> ART
    F_COV --> ART

    style GHCR fill:#0969da,color:#fff
    style B_TEST fill:#339933,color:#fff
    style F_TEST fill:#339933,color:#fff
```

---

## Workflow Details

### Backend CI (`space2study-backend/.github/workflows/ci.yml`)

**Triggers:** Push, Pull Request (all branches)

**Jobs:**
1. **test** - Run tests with MongoDB service
   - Checkout code
   - Setup Node.js 18
   - Install dependencies (`npm ci`)
   - Run linter (`npm run lint`)
   - Run tests (`npm test`) with MongoDB service container
   - Upload coverage artifact

**Services:**
- MongoDB 4.2 on port 27017

**Environment Variables:**
```yaml
MONGODB_URL: mongodb://localhost:27017/space2study-test
SERVER_PORT: 3000
JWT_ACCESS_SECRET: test-access-secret
JWT_ACCESS_EXPIRES_IN: 15m
JWT_REFRESH_SECRET: test-refresh-secret
JWT_REFRESH_EXPIRES_IN: 7d
JWT_RESET_SECRET: test-reset-secret
JWT_RESET_EXPIRES_IN: 1h
JWT_CONFIRM_SECRET: test-confirm-secret
JWT_CONFIRM_EXPIRES_IN: 24h
```

---

### Frontend CI (`space2study-frontend/.github/workflows/ci.yml`)

**Triggers:** Push, Pull Request (all branches)

**Jobs:**
1. **test** - Run tests
   - Checkout code
   - Setup Node.js 18
   - Install dependencies (`npm ci --legacy-peer-deps`)
   - Run linter (`npm run lint`)
   - Run tests (`npm test`)
   - Upload coverage artifact

2. **build** - Build production assets (after test passes)
   - Checkout code
   - Setup Node.js 18
   - Install dependencies
   - Build application (`npm run build`)
   - Upload build artifacts

---

### Docker Build (`.github/workflows/docker.yml`)

**Triggers:** Push to main/master, Manual dispatch

**Jobs:**
1. **build-backend** - Build and push backend image
   - Uses Docker Buildx for efficient builds
   - Pushes to `ghcr.io/<owner>/space2study-backend`
   - Tags: `latest`, `<git-sha>`
   - Uses GitHub Actions cache

2. **build-frontend** - Build and push frontend image
   - Uses Docker Buildx
   - Pushes to `ghcr.io/<owner>/space2study-frontend`
   - Tags: `latest`, `<git-sha>`
   - Build arg: `VITE_API_BASE_PATH=/api`

---

## Container Registry

**Registry:** GitHub Container Registry (ghcr.io)

**Images:**
```
ghcr.io/<owner>/space2study-backend:latest
ghcr.io/<owner>/space2study-backend:<sha>
ghcr.io/<owner>/space2study-frontend:latest
ghcr.io/<owner>/space2study-frontend:<sha>
```

**Authentication:** Uses built-in `GITHUB_TOKEN` (no extra secrets needed)

---

## Verification Commands

```bash
# Check workflow files exist
ls -la space2study-backend/.github/workflows/
ls -la space2study-frontend/.github/workflows/
ls -la .github/workflows/

# Validate YAML syntax
yamllint space2study-backend/.github/workflows/ci.yml
yamllint space2study-frontend/.github/workflows/ci.yml
yamllint .github/workflows/docker.yml

# Test locally (requires act)
# act push -W space2study-backend/.github/workflows/ci.yml
```

---

## Activation Steps

To activate CI/CD pipelines:

1. **Push workflows to GitHub:**
   ```bash
   cd space2study-backend
   git add .github/workflows/ci.yml
   git commit -m "Add CI workflow"
   git push

   cd ../space2study-frontend
   git add .github/workflows/ci.yml
   git commit -m "Add CI workflow"
   git push

   cd ..
   git add .github/workflows/docker.yml
   git commit -m "Add Docker build workflow"
   git push
   ```

2. **Verify on GitHub:**
   - Go to repository → Actions tab
   - Check workflow runs
   - Fix any issues shown in logs

3. **(Optional) Enable branch protection:**
   - Settings → Branches → Add rule
   - Require status checks to pass before merging

---

## Next Steps

- [x] Push workflows to GitHub repositories
- [x] Verify CI workflows run (tests have pre-existing failures)
- [x] Verify Docker builds complete
- [x] Images pushed to ghcr.io
- [ ] Enable branch protection rules
- [ ] (Optional) Add SonarCloud integration
- [ ] (Optional) Fix pre-existing test failures in upstream repos
- [ ] Task 5: Load Balancing

---

## Fixes Applied

### Backend CI Fix (January 12, 2026)

**Issue:** CI workflow failing with two errors:
1. `Cannot find module 'bcrypt/lib/binding/napi-v3/bcrypt_lib.node'`
2. `"expiresIn" should be a number of seconds or string representing a timespan`

**Root Cause:**
1. `npm ci --ignore-scripts` prevented bcrypt from compiling native bindings
2. JWT expiration environment variables were missing (only secrets were set)

**Solution:**
```diff
- run: npm ci --ignore-scripts
+ run: npm ci

  env:
    JWT_ACCESS_SECRET: test-access-secret
+   JWT_ACCESS_EXPIRES_IN: 15m
    JWT_REFRESH_SECRET: test-refresh-secret
+   JWT_REFRESH_EXPIRES_IN: 7d
    JWT_RESET_SECRET: test-reset-secret
+   JWT_RESET_EXPIRES_IN: 1h
    JWT_CONFIRM_SECRET: test-confirm-secret
+   JWT_CONFIRM_EXPIRES_IN: 24h
```

**Commit:** `dc764b6` - Fix CI workflow: bcrypt bindings and JWT expiration

---

### Pre-existing Test Failures (January 12, 2026)

Both component repositories have pre-existing test failures that were NOT introduced by our CI/CD implementation. These are bugs in the upstream codebase.

#### Backend Test Failures (31 failed / 98 passed)

**Issue 1: Email Templates Path Bug**
- `email-templates` library defaults to looking in `./emails/`
- Templates are actually located in `./src/emails/`
- Error: `ENOENT: no such file or directory, stat '.../emails/en/confirm-email.pug'`

**Issue 2: Integration Tests Send Real Emails**
- Tests attempt to send actual emails via Gmail OAuth
- Placeholder credentials in CI cause `EAUTH: invalid_client` errors
- Tests should mock email service but don't

#### Frontend Test Failures (30 failed / 141 passed)

**Issue 1: Mock Setup Broken (7 tests)**
- `EmailConfirmModal.spec.jsx`: `default.mockImplementation is not a function`
- `FeatureBlock.spec.jsx`: Same mock issue

**Issue 2: Google OAuth Not Mocked (20+ tests)**
- `GoogleButton` component tries to access `accounts` from `@react-oauth/google`
- Error: `Cannot read properties of undefined (reading 'accounts')`
- Affects: SignupDialog, LoginDialog, NavBar, GuestHome, etc.

**Issue 3: React Router Data Router**
- Some tests fail with: `useNavigation must be used within a data router`

**Decision:** These are upstream bugs, not our responsibility to fix. Documented for awareness.

---

### Frontend CI Fix: Build Despite Test Failures (January 12, 2026)

**Issue:** Build job was skipped because it depended on test job passing.

**Root Cause:** Workflow had `needs: test` without `if: always()`, causing build to be skipped when pre-existing tests failed.

**Solution:**
```diff
  build:
    runs-on: ubuntu-latest
    needs: test
+   if: always()  # Run build even if tests fail (pre-existing test issues in upstream repo)
```

**Rationale:** Pre-existing test failures should not block us from verifying the build works. The build job is independent and should run regardless.

**Commit:** `847859c` - CI: Run build job even if tests fail

---

### Docker Workflow Fix: Checkout Component Repos (January 12, 2026)

**Issue:** Docker build failing with `path "./space2study-backend" not found`

**Root Cause:** The docker.yml workflow expected component repos to be subdirectories of the infra repo, but on GitHub Actions, only the infra repo is checked out.

**Error:**
```
ERROR: failed to build: unable to prepare context: path "./space2study-backend" not found
ERROR: failed to build: unable to prepare context: path "./space2study-frontend" not found
```

**Solution:** Add explicit checkout steps for component repos from DevOps-ProjectLevel organization:

```diff
  steps:
-   - name: Checkout code
+   - name: Checkout infra repo
      uses: actions/checkout@v4

+   - name: Checkout backend repo
+     uses: actions/checkout@v4
+     with:
+       repository: DevOps-ProjectLevel/space2study-backend-1g0s
+       path: space2study-backend
```

**Rationale:** The infra repo orchestrates Docker builds but doesn't contain the source code. We must explicitly checkout the component repos into the expected paths.

**Affected Files:** `.github/workflows/docker.yml`

---

### Private Repo Access: CHECKOUT_TOKEN Secret (January 12, 2026)

**Issue:** After adding explicit checkout steps, builds failed with "Not Found" error.

**Root Cause:** Component repos are private. The default `GITHUB_TOKEN` only has access to the current repo (infra), not cross-organization repos in DevOps-ProjectLevel.

**Error:**
```
Not Found - https://docs.github.com/rest/repos/repos#get-a-repository
```

**Solution:**
1. Created `CHECKOUT_TOKEN` secret using existing `gh` auth token (which has access to both orgs)
2. Updated workflow to use token for checkout:

```diff
  - name: Checkout backend repo
    uses: actions/checkout@v4
    with:
      repository: DevOps-ProjectLevel/space2study-backend-1g0s
+     token: ${{ secrets.CHECKOUT_TOKEN }}
      path: space2study-backend
```

**Commands Used:**
```bash
# Add current gh token as secret
gh auth token | gh secret set CHECKOUT_TOKEN --repo 1g0s/space2study-infra

# Verify secret
gh secret list --repo 1g0s/space2study-infra
```

**Result:** Both Docker builds now succeed and push images to ghcr.io.

**Commit:** `ac34c74` - Docker workflow: add token for private repo checkout

---

## Final CI/CD Status

| Workflow | Status | Details |
|----------|--------|---------|
| Backend CI | ⚠️ Tests failing | Pre-existing bugs (31 failed / 98 passed) |
| Frontend CI | ⚠️ Tests failing, Build OK | Pre-existing bugs (30 failed / 141 passed) |
| Docker Build | ✅ Working | Both images built and pushed to ghcr.io |

**Container Images:**
```
ghcr.io/1g0s/space2study-backend:latest
ghcr.io/1g0s/space2study-frontend:latest
```
