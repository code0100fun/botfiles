---
name: docker-swarm
description: Docker Swarm deployment patterns covering stack deployments, networking, TLS certificates, NFS permissions, Portainer management, and GPU workloads. Use when deploying or managing Docker Swarm services.
---

# Docker Swarm Deployment Patterns

## Deployment Standards

- **All services deploy as Docker Swarm stacks** - NOT standalone containers
- Use compose files for stack definitions
- Store compose templates in version control
- Deploy with automation (Ansible playbooks, Terraform, CI/CD)

## Stack Deployment

### Deploy a Stack

```bash
docker stack deploy -c compose.yml <stack-name>
```

### Manage Stacks

```bash
docker stack ls                    # List all stacks
docker stack services <stack-name> # List services in a stack
docker stack rm <stack-name>       # Remove a stack
docker service logs <service>      # View service logs
docker service update --force <service>  # Force restart
```

## Traefik TLS Certificates

### Strategy: Individual Certs Per Service (NOT Wildcard)

- Each service gets its own Let's Encrypt certificate via DNS-01 challenge
- If some services have valid LE certs and others have "TRAEFIK DEFAULT CERT", it's likely a **Let's Encrypt rate limit** issue
- LE rate limits: 50 certs per registered domain per week
- Fix: Wait for rate limit to reset, or check Traefik logs for ACME errors

### DNS vs SSL Strategy

- **DNS Wildcard**: `*.apps.example.com -> <manager-ip>` (for automatic resolution)
- **SSL Certificates**: Individual certs per service (NOT wildcard certificate)
  - Each service requests its own certificate when deployed
  - No manual DNS updates needed (wildcard DNS handles resolution)

### Traefik Labels for SSL

When deploying services behind Traefik:

```yaml
services:
  myapp:
    deploy:
      labels:
        - "traefik.enable=true"
        - "traefik.http.routers.myapp.rule=Host(`myapp.apps.example.com`)"
        - "traefik.http.routers.myapp.tls=true"
        - "traefik.http.routers.myapp.tls.certresolver=letsencrypt"
        - "traefik.http.services.myapp.loadbalancer.server.port=8080"
```

### NFS Permissions for Containers

- NFS shares with `root_squash` prevent containers running as root from writing files
- **Solution**: Run containers as the UID that owns the files on the NFS share
- When debugging permission issues, check:
  1. Container UID (`docker exec <container> id`)
  2. File ownership on NFS (`ls -la /mnt/...`)
  3. NFS export settings

Example - running as specific user:
```yaml
services:
  traefik:
    user: "1001:991"  # Match file owner UID:GID
```

## Networking

### Overlay Networks

Use overlay networks for service-to-service communication:

```yaml
networks:
  app-network:
    driver: overlay
    attachable: true
```

### Service Discovery

Services on the same overlay network can reach each other by service name:
```
http://myservice:8080
```

## Placement Constraints

Control where services run:

```yaml
services:
  myapp:
    deploy:
      placement:
        constraints:
          - node.role == manager
          - node.labels.app.role == web
```

### Node Labels

Label nodes for placement:
```bash
docker node update --label-add app.role=web <node-id>
docker node update --label-add gpu.type=nvidia <node-id>
```

## GPU Workloads

For services requiring GPU access:

```yaml
services:
  gpu-service:
    deploy:
      placement:
        constraints:
          - node.labels.gpu.type == nvidia
      resources:
        reservations:
          devices:
            - capabilities: [gpu]
```

- Use overlay networking + Traefik labels (no host-mode ports) for GPU workloads behind a reverse proxy
- Ensure NVIDIA runtime is configured in `/etc/docker/daemon.json` on GPU nodes

## Portainer Management

### Agent Restart After Docker Restarts

If Portainer shows "Unable to connect to Docker environment" after Docker restarts:
- **Cause**: Docker daemon restart causes agent containers to get new IDs, stale overlay network connections
- **Fix**: Force restart Portainer agents:
  ```bash
  docker service update --force portainer_agent
  ```

## Health Checks

Always include health checks in compose files:

```yaml
services:
  myapp:
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
```

## Rolling Updates

Configure update behavior:

```yaml
services:
  myapp:
    deploy:
      update_config:
        parallelism: 1
        delay: 10s
        failure_action: rollback
        order: start-first
      rollback_config:
        parallelism: 1
        delay: 5s
```

## Debugging

```bash
# Check service status
docker service ps <service> --no-trunc

# View service logs
docker service logs -f <service>

# Inspect service config
docker service inspect <service>

# Check node status
docker node ls

# Check what's running on a node
docker node ps <node-id>
```
