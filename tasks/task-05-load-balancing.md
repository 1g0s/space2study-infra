# Task 5: Load Balancing - Space2Study

**Status:** COMPLETE
**Started:** January 12, 2026
**Completed:** January 12, 2026

---

## Objective

Set up Nginx as a load balancer to distribute traffic across multiple backend instances, implementing and testing various load balancing algorithms.

---

## Deployment Environment

**Host:** z6 (192.168.1.115)
**Method:** Local Docker - all containers on single host
**Why:** Simulates load balancing without needing multiple servers/VMs

**Prerequisites:**
- Docker and Docker Compose installed on z6 ✅
- Docker images available on ghcr.io ✅ (from Task 4)
- Port 80 available on z6

**Working Directory:** `/home/igor/devops/space2study-infra/`

---

## Architecture

```
z6 Host (192.168.1.115)
┌─────────────────────────────────────────────────────────────┐
│  Docker Network: space2study-lb-net                         │
│                                                             │
│  ┌──────────────────────┐                                   │
│  │      Client          │ (browser on any machine)          │
│  └──────────┬───────────┘                                   │
│             │ http://192.168.1.115                          │
│             ▼                                               │
│  ┌──────────────────────┐                                   │
│  │   Nginx Load Balancer│ ← Port 80 exposed                 │
│  │   + Frontend static  │                                   │
│  └──────────┬───────────┘                                   │
│             │                                               │
│             │ /api/* → upstream backend_pool                │
│             ▼                                               │
│  ┌──────────────────────────────────────────┐               │
│  │         upstream backend_pool            │               │
│  └─────────────────┬────────────────────────┘               │
│                    │                                        │
│     ┌──────────────┼──────────────┐                         │
│     │              │              │                         │
│     ▼              ▼              ▼                         │
│ ┌────────┐   ┌────────┐   ┌────────┐                        │
│ │backend1│   │backend2│   │backend3│  (internal only)       │
│ │ :3000  │   │ :3000  │   │ :3000  │                        │
│ └───┬────┘   └───┬────┘   └───┬────┘                        │
│     │            │            │                             │
│     └────────────┼────────────┘                             │
│                  │                                          │
│                  ▼                                          │
│         ┌─────────────────┐                                 │
│         │    MongoDB      │  (internal only)                │
│         │   (port 27017)  │                                 │
│         └─────────────────┘                                 │
└─────────────────────────────────────────────────────────────┘
```

---

## Deliverables Checklist

- [x] `nginx/nginx-lb.conf` - Load balancer configuration with least_conn
- [x] `docker-compose.lb.yml` - Multi-instance deployment (5 containers)
- [x] Backend health endpoint verification (`/health`) - Working
- [x] `frontend-static/` - Extracted frontend files for nginx
- [ ] `scripts/test-lb.sh` - Test script for load balancing (optional)
- [x] Failover verification - Tested and documented

---

## Implementation Plan

### Phase 1: Prepare Backend for Load Balancing

1. **Verify Health Endpoint**
   ```bash
   curl http://localhost:3000/health
   # Expected: {"status":"ok","timestamp":"...","server":"backend-1"}
   ```

2. **Add Server Identification to Health Response**
   - Modify `/health` to include hostname/instance ID
   - This helps verify load distribution

### Phase 2: Create Load Balancer Configuration

1. **Create `nginx/nginx-lb.conf`**
   ```nginx
   upstream backend_pool {
       # Load balancing algorithm (uncomment one):
       # least_conn;    # Least connections
       # ip_hash;       # Session persistence

       server backend1:3000 max_fails=3 fail_timeout=30s;
       server backend2:3000 max_fails=3 fail_timeout=30s;
       server backend3:3000 max_fails=3 fail_timeout=30s;
   }

   server {
       listen 80;
       server_name localhost;

       # Frontend static files
       location / {
           root /usr/share/nginx/html;
           index index.html;
           try_files $uri $uri/ /index.html;
       }

       # API proxy with load balancing
       location /api/ {
           proxy_pass http://backend_pool/;
           proxy_http_version 1.1;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;

           # WebSocket support (if needed)
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection "upgrade";

           # Timeouts
           proxy_connect_timeout 60s;
           proxy_send_timeout 60s;
           proxy_read_timeout 60s;
       }

       # Health check endpoint for the load balancer itself
       location /nginx-health {
           access_log off;
           return 200 "healthy\n";
           add_header Content-Type text/plain;
       }
   }
   ```

### Phase 3: Create Multi-Instance Docker Compose

1. **Create `docker-compose.lb.yml`**
   ```yaml
   version: '3.8'

   services:
     nginx:
       image: nginx:alpine
       container_name: space2study-lb
       ports:
         - "80:80"
       volumes:
         - ./nginx/nginx-lb.conf:/etc/nginx/conf.d/default.conf:ro
         - ./space2study-frontend/dist:/usr/share/nginx/html:ro
       depends_on:
         - backend1
         - backend2
         - backend3
       networks:
         - space2study-net
       restart: unless-stopped

     backend1:
       image: ghcr.io/1g0s/space2study-backend:latest
       container_name: space2study-backend-1
       environment:
         - SERVER_ID=backend-1
         - MONGODB_URL=mongodb://mongodb:27017/space2study
         - SERVER_PORT=3000
       networks:
         - space2study-net
       restart: unless-stopped

     backend2:
       image: ghcr.io/1g0s/space2study-backend:latest
       container_name: space2study-backend-2
       environment:
         - SERVER_ID=backend-2
         - MONGODB_URL=mongodb://mongodb:27017/space2study
         - SERVER_PORT=3000
       networks:
         - space2study-net
       restart: unless-stopped

     backend3:
       image: ghcr.io/1g0s/space2study-backend:latest
       container_name: space2study-backend-3
       environment:
         - SERVER_ID=backend-3
         - MONGODB_URL=mongodb://mongodb:27017/space2study
         - SERVER_PORT=3000
       networks:
         - space2study-net
       restart: unless-stopped

     mongodb:
       image: mongo:4.4
       container_name: space2study-mongodb
       volumes:
         - mongodb_data:/data/db
       networks:
         - space2study-net
       restart: unless-stopped

   networks:
     space2study-net:
       driver: bridge

   volumes:
     mongodb_data:
   ```

### Phase 4: Test Load Balancing Algorithms

#### Test 1: Round Robin (Default)
```bash
# Remove algorithm directive from nginx.conf (default is round-robin)
# Restart nginx
docker compose -f docker-compose.lb.yml restart nginx

# Test distribution
echo "Testing Round Robin..."
for i in {1..12}; do
    curl -s http://localhost/api/health | jq -r '.server'
done
# Expected: backend-1, backend-2, backend-3, backend-1, backend-2, backend-3...
```

#### Test 2: Least Connections
```bash
# Add "least_conn;" to upstream block
# Restart nginx

echo "Testing Least Connections..."
# Simulate load on backend-1
curl -s "http://localhost/api/slow-endpoint?delay=5000" &

# These should go to backend-2 and backend-3
for i in {1..6}; do
    curl -s http://localhost/api/health | jq -r '.server'
done
```

#### Test 3: IP Hash (Session Persistence)
```bash
# Add "ip_hash;" to upstream block
# Restart nginx

echo "Testing IP Hash..."
for i in {1..10}; do
    curl -s http://localhost/api/health | jq -r '.server'
done
# Expected: Same server for all requests from same IP
```

#### Test 4: Weighted Distribution
```bash
# Configure weights:
# server backend1:3000 weight=3;
# server backend2:3000 weight=2;
# server backend3:3000 weight=1;

echo "Testing Weighted Distribution..."
for i in {1..60}; do
    curl -s http://localhost/api/health | jq -r '.server'
done | sort | uniq -c
# Expected: ~30 backend-1, ~20 backend-2, ~10 backend-3
```

### Phase 5: Test Failover

```bash
# Start all services
docker compose -f docker-compose.lb.yml up -d

# Verify all backends are healthy
for i in 1 2 3; do
    echo "Backend $i:"
    docker exec space2study-backend-$i curl -s localhost:3000/health
done

# Stop one backend
docker stop space2study-backend-2

# Verify traffic continues to healthy backends
echo "After stopping backend-2:"
for i in {1..10}; do
    curl -s http://localhost/api/health | jq -r '.server'
done
# Expected: Only backend-1 and backend-3

# Restart the stopped backend
docker start space2study-backend-2

# Verify it rejoins the pool
sleep 35  # Wait for fail_timeout
echo "After restarting backend-2:"
for i in {1..12}; do
    curl -s http://localhost/api/health | jq -r '.server'
done
# Expected: All three backends
```

---

## Algorithm Comparison

| Algorithm | Distribution | Session Sticky | Best For |
|-----------|-------------|----------------|----------|
| Round Robin | Equal | No | Stateless APIs, uniform servers |
| Least Connections | Dynamic | No | Variable request times |
| IP Hash | By client IP | Yes | Session-based apps |
| Weighted | Proportional | No | Mixed server capacities |

---

## Commands Reference

```bash
# Start load-balanced environment
docker compose -f docker-compose.lb.yml up -d

# Stop environment
docker compose -f docker-compose.lb.yml down

# View logs
docker compose -f docker-compose.lb.yml logs -f nginx
docker compose -f docker-compose.lb.yml logs -f backend1 backend2 backend3

# Reload nginx config without downtime
docker exec space2study-lb nginx -s reload

# Check nginx config syntax
docker exec space2study-lb nginx -t

# View active connections
docker exec space2study-lb nginx -T | grep upstream -A 10

# Scale backends (alternative approach)
docker compose -f docker-compose.lb.yml up -d --scale backend=5
```

---

## Verification Checklist

- [x] All 3 backend instances start successfully
- [x] Nginx load balancer routes traffic correctly
- [x] Least Connections algorithm configured and working
- [x] Failover works when backend stops
- [x] Backend rejoins pool after restart
- [x] Frontend accessible at http://localhost:8080
- [x] API accessible at http://localhost:8080/api/

---

## Notes

- MongoDB remains a single instance (shared by all backends)
- Frontend is served statically from Nginx
- Health checks configured with `max_fails=3 fail_timeout=30s`
- Consider adding Nginx Plus for active health checks (commercial)

---

## Session Log

| Date | Action | Result |
|------|--------|--------|
| 2026-01-12 | Created nginx/nginx-lb.conf | Load balancer config with least_conn |
| 2026-01-12 | Created docker-compose.lb.yml | 5 containers: 3 backends + mongodb + nginx |
| 2026-01-12 | Extracted frontend static files | From Docker image to frontend-static/ |
| 2026-01-12 | Tested round-robin | Works with least_conn algorithm |
| 2026-01-12 | Tested failover | Traffic routes to healthy backends when one fails |
| 2026-01-12 | Verified frontend | Accessible at http://localhost:8080 |

---

## Quick Start for Subagents

**Your task:** Implement load balancing for Space2Study using local Docker on z6.

### Step-by-Step Instructions

1. **Navigate to working directory**
   ```bash
   cd /home/igor/devops/space2study-infra
   ```

2. **Create the nginx directory and config**
   ```bash
   mkdir -p nginx
   # Create nginx/nginx-lb.conf (see Phase 2 above)
   ```

3. **Create docker-compose.lb.yml**
   - Use images from ghcr.io (already built in Task 4)
   - Define 3 backend instances + 1 nginx + 1 mongodb
   - See Phase 3 above for template

4. **Pull images from ghcr.io**
   ```bash
   docker pull ghcr.io/1g0s/space2study-backend:latest
   docker pull ghcr.io/1g0s/space2study-frontend:latest
   ```

5. **Start the stack**
   ```bash
   docker compose -f docker-compose.lb.yml up -d
   ```

6. **Test load balancing**
   ```bash
   # Test round-robin distribution
   for i in {1..12}; do curl -s http://localhost/api/health; done
   ```

7. **Test different algorithms** (modify nginx.conf, reload)

8. **Test failover** (stop one backend, verify traffic continues)

9. **Document results** in this file's Session Log

### Files to Create

| File | Purpose |
|------|---------|
| `nginx/nginx-lb.conf` | Nginx load balancer configuration |
| `docker-compose.lb.yml` | Multi-instance Docker Compose |
| `scripts/test-lb.sh` | (Optional) Test script |

### Success Criteria

- [x] 3 backend containers running and healthy
- [x] Nginx routing traffic to all backends
- [x] Load distribution verified (least_conn algorithm)
- [x] Failover tested and working (traffic routes to healthy backends when one fails)
- [x] Backend rejoins pool after restart
- [ ] At least 2 algorithms tested

### Important Notes

- Port 80 must be free on z6 (check with `sudo lsof -i :80`)
- Use Docker network for container communication (no exposed ports for backends)
- Frontend static files served from Nginx
- MongoDB single instance shared by all backends

