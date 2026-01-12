# Space2Study Infrastructure

DevOps infrastructure and orchestration for the Space2Study project.

## Project Overview

SpaceToStudy is a platform where experts share knowledge and students can learn from the best. Find training courses, tutors, or students and receive feedback.

## Component Repositories

| Component | Repository | Technology |
|-----------|------------|------------|
| Backend | [space2study-backend](https://github.com/DevOps-ProjectLevel/space2study-backend-1g0s) | Node.js 18, Express |
| Frontend | [space2study-frontend](https://github.com/DevOps-ProjectLevel/space2study-frontend-1g0s) | React 17, Vite |

## Infrastructure Files

```
├── docker-compose.yml          # Development environment
├── docker-compose.prod.yml     # Production deployment
├── .env.prod.example           # Environment template
├── .github/workflows/          # CI/CD workflows
│   └── docker.yml              # Docker build & push
├── epic.md                     # Project tasks & progress
└── tasks/                      # Task completion reports
    ├── task-01-setup-webapp.md
    ├── task-02-containerization.md
    └── task-04-cicd.md
```

## Quick Start

### Development
```bash
# Clone component repos
git clone git@github.com:DevOps-ProjectLevel/space2study-backend-1g0s.git space2study-backend
git clone git@github.com:DevOps-ProjectLevel/space2study-frontend-1g0s.git space2study-frontend

# Start MongoDB
docker compose up -d

# Run backend (terminal 1)
cd space2study-backend && npm install && npm start

# Run frontend (terminal 2)
cd space2study-frontend && npm install && npm start
```

### Production
```bash
# Build images
docker build -t space2study-backend:prod ./space2study-backend
docker build -t space2study-frontend:prod ./space2study-frontend

# Create environment file
cp .env.prod.example .env.prod
# Edit .env.prod with real values

# Run production stack
docker compose -f docker-compose.prod.yml up -d
```

## Task Progress

| Task | Status | Tag |
|------|--------|-----|
| 1. Setup Webapp | ✅ Complete | `task-1` |
| 2. Containerization | ✅ Complete | `task-2` |
| 3. Infrastructure as Code | ⏭️ Skipped | - |
| 4. CI/CD Pipeline | ✅ Complete | `task-4` |
| 5. Load Balancing | ⬜ Pending | - |
| 6. Automation | ⬜ Pending | - |
| 7. Kubernetes | ⬜ Pending | - |
| 8. Cloud Migration | ⬜ Pending | - |

## Container Images

Production images are pushed to GitHub Container Registry:

```
ghcr.io/<owner>/space2study-backend:latest
ghcr.io/<owner>/space2study-frontend:latest
```

## Documentation

- [epic.md](epic.md) - Full project overview and task details
- [tasks/](tasks/) - Individual task completion reports
