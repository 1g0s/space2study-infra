Space2Study project
Project Overview
SpaceToStudy project is a platform where experts in various fields share their knowledge and students can learn from the best. Here you can find the proper training course, find a tutor, or find students and receive feedback from them.

Here is two part: back-end and front-end

Project built
Node.js 18.14.0

React 17.0.2

## Repositories

| Repository | Purpose | URL |
|------------|---------|-----|
| **space2study-infra** | Infrastructure & DevOps | `git@github.com:1g0s/space2study-infra.git` |
| **space2study-backend** | Node.js Backend API | `git@github.com:DevOps-ProjectLevel/space2study-backend-1g0s.git` |
| **space2study-frontend** | React Frontend | `git@github.com:DevOps-ProjectLevel/space2study-frontend-1g0s.git` |

---

## Tasks

### Core Tasks (in order)
1. Setup a Webapp
2. Deploying a Containerized Web Application
3. Implement Infrastructure as Code (Vagrant + Ansible)
4. Implement a Continuous Integration/Continuous Delivery
5. Setup Load Balancing for Webapp
   - Deploy couple servers
   - Set up Apache or Nginx as a load balancer
   - Try different load-balancing algorithms and options
6. Implement Automatisation Setup a Webapp
7. Orchestration Web Application via k8s
8. Migrate an Application to the Cloud

### Additional Tasks
- Set up monitoring tools for application performance and infrastructure health
- Configure logging mechanisms for tracking application and system logs
- Monitor resource usage and plan for scalability
- Implement CCI (Continuous Code Inspection)
- Artifact Management

---

## Implementation Plan

### Project Architecture Summary

| Component | Technology | Port |
|-----------|-----------|------|
| **space2study-backend** | Node.js 18.14.0, Express 4.17.1 | 3000 |
| **space2study-frontend** | React 17.0.2, Vite, TypeScript | 3000 (dev) / 80 (prod) |
| **Database** | MongoDB 4.2+ | 27017 |
| **Storage** | Azure Blob Storage | - |

### Infrastructure Repository Structure

```
space2study-infra/
├── .github/workflows/
│   └── docker.yml           # Docker build & push to ghcr.io
├── docker-compose.yml       # Development environment
├── docker-compose.prod.yml  # Production deployment
├── .env.prod.example        # Environment template
├── epic.md                  # This file
├── README.md                # Quick start guide
└── tasks/                   # Task completion reports
    ├── task-01-setup-webapp.md
    ├── task-02-containerization.md
    └── task-04-cicd.md
```

### Component Repos Include
- ✅ SonarCloud integration (both repos)
- ✅ Jest/Vitest testing with coverage
- ✅ ESLint + Prettier code quality
- ✅ Dockerfiles (backend, frontend)
- ✅ CI workflows (.github/workflows/ci.yml)

---

### Task 1: Setup a Webapp

**Objective:** Get both components running locally

**Steps:**
1. **Database Setup**
   ```bash
   # Option 1: Docker
   docker run -d --name mongodb -p 27017:27017 mongo:4.2

   # Option 2: Use existing MongoDB on z6
   # mongodb://192.168.1.115:27017
   ```

2. **Backend Service**
   ```bash
   cd space2study-backend
   cp .env.example .env   # Create env file if needed
   npm install
   npm start              # Runs with nodemon
   ```

3. **Frontend Application**
   ```bash
   cd space2study-frontend
   cp .env.example .env
   npm install
   npm run start          # Vite dev server on port 3000
   ```

**Environment Variables Required:**

**Backend (.env):**
```bash
# Core
MONGODB_URL=mongodb://localhost:27017/space2study
SERVER_PORT=3000
SERVER_URL=http://localhost:3000
CLIENT_URL=http://localhost:3000
COOKIE_DOMAIN=localhost

# JWT Secrets (4 token types)
JWT_ACCESS_SECRET=<secret>
JWT_ACCESS_EXPIRES_IN=15m
JWT_REFRESH_SECRET=<secret>
JWT_REFRESH_EXPIRES_IN=7d
JWT_RESET_SECRET=<secret>
JWT_RESET_EXPIRES_IN=1h
JWT_CONFIRM_SECRET=<secret>
JWT_CONFIRM_EXPIRES_IN=24h

# Email (Gmail OAuth2)
MAIL_USER=<gmail>
GMAIL_CLIENT_ID=<client-id>
GMAIL_CLIENT_SECRET=<client-secret>
GMAIL_REFRESH_TOKEN=<refresh-token>
GMAIL_REDIRECT_URI=https://developers.google.com/oauthplayground

# Azure Storage
STORAGE_ACCOUNT=<account-name>
ACCESS_KEY=<access-key>
AZURE_HOST=https://<account>.blob.core.windows.net
```

**Frontend (.env):**
```bash
VITE_API_BASE_PATH=http://localhost:3000
```

**Deliverables:**
- [x] Local development environment running
- [x] Backend API accessible at http://localhost:3001
- [x] Frontend accessible at http://localhost:3000 (dev port)
- [ ] API documentation at /api/docs (Swagger)

**Status:** COMPLETED (January 6, 2026)

> **Full Report:** [tasks/task-01-setup-webapp.md](tasks/task-01-setup-webapp.md)


---

### Task 2: Deploying a Containerized Web Application

**Objective:** Containerize both services with production-ready Dockerfiles

**Deliverables:**
- [x] Dockerfile (backend) - Multi-stage Node.js production build
- [x] Dockerfile (frontend) - Multi-stage build with Nginx
- [x] nginx.conf - SPA routing, API proxy, security headers
- [x] .dockerignore files - For both repos
- [x] docker-compose.prod.yml - Production configuration
- [x] .env.prod.example - Environment template
- [x] Health endpoint - `/health` route in backend

**Status:** ✅ VERIFIED COMPLETE (January 11, 2026)

> **Full Report:** [tasks/task-02-containerization.md](tasks/task-02-containerization.md)

---

### Task 3: Infrastructure as Code (Vagrant + Ansible)

**Objective:** Provision local development/test infrastructure using Infrastructure as Code tools

**Requirements:**
- Vagrant for VM provisioning (VirtualBox provider)
- Ansible for configuration management and application deployment
- Reproducible local environment that mirrors production

**Deliverables:**
- [ ] Vagrantfile - Define VM(s) for the application stack
- [ ] Ansible inventory - Host definitions
- [ ] Ansible playbooks - Provision and deploy application
  - [ ] Install Docker and dependencies
  - [ ] Deploy MongoDB container
  - [ ] Deploy backend container
  - [ ] Deploy frontend container
- [ ] README for IaC setup instructions

**Infrastructure:**
| VM | Purpose | Resources |
|----|---------|-----------|
| space2study-vm | Full stack deployment | 2 CPU, 4GB RAM |

**Status:** ⬜ Not Started

---

### Task 4: CI/CD Pipeline

**Objective:** Implement automated CI/CD pipelines with GitHub Actions

**Deliverables:**
- [x] Backend CI workflow - Jest tests, lint, coverage
- [x] Frontend CI workflow - Vitest tests, lint, build
- [x] Docker build workflow - Build & push to ghcr.io

**Status:** IMPLEMENTED (January 11, 2026)

> **Full Report:** [tasks/task-04-cicd.md](tasks/task-04-cicd.md)

**Known Issues (Pre-existing in Upstream Repo):**

Both repositories have pre-existing test failures that are NOT issues we introduced:

**Backend (31 failed / 98 passed):**
- Email templates path bug: `email-templates` library looks in `./emails/` but templates are in `./src/emails/`
- Integration tests try to send real emails via Gmail OAuth with placeholder credentials

**Frontend (30 failed / 141 passed):**
- `EmailConfirmModal`: `default.mockImplementation is not a function` (5 tests)
- `GoogleButton/OAuth`: `Cannot read properties of undefined (reading 'accounts')` (20+ tests)
- React Router: `useNavigation must be used within a data router`

**Workaround Applied:**
- Frontend CI workflow updated with `if: always()` on build job to run build even when tests fail
- This ensures we can verify the build works despite pre-existing test issues

---

### Progress Tracking

| Task | Status | Notes |
|------|--------|-------|
| 1. Setup Webapp | ✅ Completed | MongoDB (Docker), Backend/Frontend (native Node.js) |
| 2. Containerization | ✅ Verified | All images built, tested, all 3 containers healthy |
| 3. Infrastructure as Code | ⬜ Not Started | Vagrant + Ansible local provisioning |
| 4. CI/CD Pipeline | ✅ Implemented | GitHub Actions + ghcr.io |
| 5. Load Balancing | ⬜ Not Started | |
| 6. Automation | ⬜ Not Started | |
| 7. Kubernetes | ⬜ Not Started | |
| 8. Cloud Migration | ⬜ Not Started | |
| Monitoring | ⬜ Not Started | Winston configured |
| Logging | ⬜ Not Started | Winston configured |
| CCI | ⬜ Not Started | SonarCloud configured |
| Artifact Management | ⬜ Not Started | |

---


## Session Log

| Date | Task | Summary |
|------|------|---------|
| 2026-01-06 | Task 1 | Set up MongoDB Docker, Node.js backend/frontend natively |
| 2026-01-06 | Task 2 | Created Dockerfiles, nginx.conf, health endpoint, docker-compose.prod.yml |
| 2026-01-11 | Task 2 | Fixed bcrypt native binding, swagger-settings; verified full stack (3 containers healthy) |
| 2026-01-11 | Task 4 | Created GitHub Actions CI/CD workflows for backend, frontend, and Docker builds |
| 2026-01-12 | Infra | Created infrastructure repo with git history and tags for Tasks 1, 2, 4 |
| 2026-01-12 | Task 2,4 | Pushed Dockerfiles, CI workflows to component repos (DevOps-ProjectLevel) |
| 2026-01-12 | Task 4 | Fixed backend CI: removed --ignore-scripts for bcrypt, added JWT expiration env vars |
| 2026-01-12 | Task 4 | Investigated CI failures: identified pre-existing test bugs in upstream repos |
| 2026-01-12 | Task 4 | Frontend CI: added `if: always()` to build job to run despite test failures |
| 2026-01-12 | Task 4 | Docker workflow: fixed to checkout component repos from DevOps-ProjectLevel org |

---

## Task Completion Reports

All detailed task completion reports are maintained in the `tasks/` directory:

| Task | Report |
|------|--------|
| Task 1: Setup Webapp | [tasks/task-01-setup-webapp.md](tasks/task-01-setup-webapp.md) |
| Task 2: Containerization | [tasks/task-02-containerization.md](tasks/task-02-containerization.md) |
| Task 4: CI/CD Pipeline | [tasks/task-04-cicd.md](tasks/task-04-cicd.md) |

