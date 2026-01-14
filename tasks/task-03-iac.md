# Task 3: Infrastructure as Code - Space2Study

**Status:** Completed
**Started:** January 14, 2026
**Completed:** January 14, 2026

---

## Objective

Provision a clean Ubuntu VM using Vagrant and deploy the full Space2Study application stack using Ansible. This demonstrates Infrastructure as Code principles with reproducible, automated deployments.

---

## Deployment Environment

**Host:** z6 (192.168.1.115)
**Hypervisor:** VirtualBox
**Provisioner:** Vagrant + Ansible (ansible_local)
**Target VM:** Ubuntu 22.04 LTS (Jammy)

---

## VM Specifications

| Property | Value |
|----------|-------|
| Name | space2study-vm |
| IP Address | 192.168.10.10 |
| RAM | 4GB |
| CPUs | 2 |
| OS | Ubuntu 22.04 (ubuntu/jammy64) |
| Containers | 5 (nginx + 3 backends + mongodb) |

---

## Architecture

```
z6 Host (192.168.1.115)
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  VirtualBox                                                         │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │  space2study-vm                                               │  │
│  │  OS: Ubuntu 22.04 LTS                                         │  │
│  │  IP: 192.168.10.10 (host-only network)                        │  │
│  │  Resources: 2 CPU, 4GB RAM                                    │  │
│  │                                                               │  │
│  │  ┌─────────────────────────────────────────────────────────┐  │  │
│  │  │  Docker                                                 │  │  │
│  │  │                                                         │  │  │
│  │  │  ┌─────────────┐                                        │  │  │
│  │  │  │   nginx     │ :80 (exposed)                          │  │  │
│  │  │  │   (LB)      │                                        │  │  │
│  │  │  └──────┬──────┘                                        │  │  │
│  │  │         │                                               │  │  │
│  │  │  ┌──────┴──────┬─────────────┐                          │  │  │
│  │  │  │             │             │                          │  │  │
│  │  │  ▼             ▼             ▼                          │  │  │
│  │  │ backend1    backend2    backend3                        │  │  │
│  │  │ :3000       :3000       :3000                           │  │  │
│  │  │  │             │             │                          │  │  │
│  │  │  └─────────────┼─────────────┘                          │  │  │
│  │  │                │                                        │  │  │
│  │  │                ▼                                        │  │  │
│  │  │           ┌─────────┐                                   │  │  │
│  │  │           │ mongodb │ :27017                            │  │  │
│  │  │           └─────────┘                                   │  │  │
│  │  └─────────────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                                                                     │
│  Access: http://192.168.10.10                                       │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Deliverables Checklist

- [x] `vagrant/Vagrantfile` - VM definition with ansible_local provisioner
- [x] `ansible/inventory.yml` - Host inventory (localhost for ansible_local)
- [x] `ansible/playbook.yml` - Main playbook with 3 roles
- [x] `ansible/ansible.cfg` - Ansible configuration
- [x] `ansible/requirements.yml` - Galaxy collection requirements
- [x] `ansible/roles/common/` - Base system setup
- [x] `ansible/roles/docker/` - Docker installation
- [x] `ansible/roles/app/` - Application deployment
- [x] `ansible/group_vars/all.yml` - Variables

---

## Directory Structure

```
space2study-infra/
├── vagrant/
│   └── Vagrantfile
└── ansible/
    ├── ansible.cfg
    ├── inventory.yml
    ├── playbook.yml
    ├── requirements.yml
    ├── group_vars/
    │   └── all.yml
    └── roles/
        ├── common/
        │   └── tasks/main.yml
        ├── docker/
        │   └── tasks/main.yml
        └── app/
            ├── tasks/main.yml
            ├── templates/
            │   └── env.j2
            └── files/
                ├── docker-compose.lb.yml
                └── nginx-lb.conf
```

---

## Components

### Vagrantfile
- Uses `ansible_local` provisioner (runs inside VM - no Ansible needed on host)
- Syncs local repos via shared folders (no GitHub clone needed)
- Configures VirtualBox with 4GB RAM, 2 CPUs
- Sets up private network with static IP (192.168.10.10)

### Ansible Roles

1. **common** - Base system setup
   - Updates apt cache
   - Installs base packages (git, curl, python3-pip, etc.)
   - Sets timezone to UTC
   - Creates application directory

2. **docker** - Docker installation
   - Removes old Docker packages
   - Adds Docker GPG key and repository
   - Installs Docker CE, CLI, containerd, compose plugin
   - Adds vagrant user to docker group
   - Installs Docker Compose standalone (v2.24.0)

3. **app** - Application deployment
   - Builds backend Docker image from local source
   - Builds frontend Docker image from local source
   - Pulls MongoDB 4.4 and nginx:alpine images
   - Copies nginx load balancer config
   - Copies frontend static files
   - Creates .env file from Jinja2 template
   - Deploys with docker-compose
   - Verifies health endpoints

---

## Usage

### Start VM and Deploy
```bash
cd /home/igor/devops/space2study-infra/vagrant
vagrant up
```

### Re-provision Existing VM
```bash
vagrant provision
```

### Reload VM (Apply Vagrantfile Changes)
```bash
vagrant reload --provision
```

### Access Application
```bash
# Frontend
curl http://192.168.10.10/

# API Health
curl http://192.168.10.10/api/health

# Load Balancer Status
curl http://192.168.10.10/lb-status

# Nginx Health
curl http://192.168.10.10/nginx-health
```

### Destroy and Recreate (Full Test)
```bash
vagrant destroy -f && vagrant up
```

### SSH into VM
```bash
vagrant ssh
```

### Check Container Status (inside VM)
```bash
vagrant ssh -c "docker ps"
```

---

## Verification Results

### Container Status
```
NAMES                    STATUS
space2study-lb           Up (healthy)
space2study-backend-3    Up (healthy)
space2study-backend-2    Up (healthy)
space2study-backend-1    Up (healthy)
space2study-lb-mongodb   Up (healthy)
```

### Endpoint Tests
| Endpoint | Status |
|----------|--------|
| http://192.168.10.10/ | 200 OK (Frontend) |
| http://192.168.10.10/api/health | 200 OK (API) |
| http://192.168.10.10/nginx-health | 200 OK |
| http://192.168.10.10/lb-status | 200 OK |

---

## Notes

### VirtualBox Network Configuration
The VM uses IP 192.168.10.10 (within the allowed range in `/etc/vbox/networks.conf`).

Current allowed ranges:
```
* 192.168.10.0/24
```

To use 192.168.56.x range, add to `/etc/vbox/networks.conf`:
```
* 192.168.56.0/24
```

### Local Source Repos
The Ansible playbook uses locally cloned repositories via Vagrant shared folders instead of cloning from GitHub. This requires the following directories to exist:
- `space2study-backend/`
- `space2study-frontend/`
- `nginx/`
- `frontend-static/`

### ansible_local Provisioner
Uses ansible_local instead of ansible provisioner:
- Ansible runs inside the VM (no Ansible needed on host)
- More portable - works without Ansible installed on host
- Installs Ansible automatically via pip3
- Galaxy collections installed inside VM

### Provisioning Time
- First run: ~10-15 minutes (downloads box, builds images)
- Subsequent provisions: ~2-3 minutes (uses cached images)

---

## Files Created

| File | Description |
|------|-------------|
| vagrant/Vagrantfile | Vagrant configuration with ansible_local |
| ansible/ansible.cfg | Ansible configuration |
| ansible/inventory.yml | Inventory for ansible_local (localhost) |
| ansible/playbook.yml | Main playbook with 3 roles |
| ansible/requirements.yml | Galaxy collection requirements |
| ansible/group_vars/all.yml | Global variables |
| ansible/roles/common/tasks/main.yml | Common role tasks |
| ansible/roles/docker/tasks/main.yml | Docker role tasks |
| ansible/roles/app/tasks/main.yml | App role tasks |
| ansible/roles/app/templates/env.j2 | Environment file template |
| ansible/roles/app/files/docker-compose.lb.yml | Docker Compose config |
| ansible/roles/app/files/nginx-lb.conf | Nginx LB config |

---

## Session Log

| Date | Action | Result |
|------|--------|--------|
| 2026-01-14 | Created Vagrantfile | Success |
| 2026-01-14 | Created Ansible directory structure | Success |
| 2026-01-14 | Created Ansible roles (common, docker, app) | Success |
| 2026-01-14 | Fixed VirtualBox network range issue | Changed IP to 192.168.10.10 |
| 2026-01-14 | Fixed ansible_local pip_install_cmd | Changed to install_mode: pip3 |
| 2026-01-14 | Fixed GitHub clone failure | Used local repos via shared folders |
| 2026-01-14 | Fixed missing mongo image | Added pull tasks for base images |
| 2026-01-14 | vagrant up completed | All 5 containers running |
| 2026-01-14 | Verified endpoints | All returning 200 OK |
| 2026-01-14 | Task completed | Success |
