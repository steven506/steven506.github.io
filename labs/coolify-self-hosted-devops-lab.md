# Coolify Self-Hosted DevOps Lab

## Overview

This lab documents the installation and validation of Coolify on a self-hosted Ubuntu VM running on VMware Fusion. The goal was to understand Coolify as a self-hosted platform for deploying applications, managing Docker Compose stacks, routing traffic through Traefik, and validating PostgreSQL database persistence.

## Lab Environment

| Component | Details |
|---|---|
| Host Machine | MacBook Pro Apple Silicon |
| Hypervisor | VMware Fusion |
| Guest OS | Ubuntu Server ARM64 |
| VM Network Mode | VMware NAT / Share with my Mac |
| Access Method | SSH from macOS |
| Platform | Coolify v4.1.2 |
| Runtime | Docker |
| Reverse Proxy | Traefik |
| Database | PostgreSQL 16 Alpine |
| Database UI | Adminer |

## VM Preparation

The Ubuntu VM was configured with a static IP address to avoid losing SSH access after reboots.

Final VM resources:

```text
CPU: 4 cores
RAM: 4.7 GiB
Disk: 58 GB root filesystem
```

Validation commands:

```bash
free -h
df -h /
nproc
lsblk
```

## Coolify Installation

Coolify was installed using the official installation script:

```bash
curl -fsSL https://cdn.coollabs.io/coolify/install.sh | sudo bash
```

After installation, the Coolify containers were validated with:

```bash
docker ps
```

Core Coolify containers included:

```text
coolify
coolify-db
coolify-redis
coolify-proxy
coolify-realtime
coolify-sentinel
```

## Accessing Coolify from macOS

Because the VM was running behind VMware NAT, direct browser access from macOS to the VM IP was not always available. SSH port forwarding was used to access Coolify locally.

```bash
ssh -L 8000:localhost:8000 ubuntu-machine
```

Coolify was accessed from the browser at:

```text
http://localhost:8000
```

## Lab 1: Docker Image Deployment

The first deployment used a simple public Docker image:

```text
traefik/whoami
```

### Purpose

- Validate basic Coolify application deployment
- Understand deployment from a public Docker image
- Test Traefik reverse proxy routing
- Confirm application access using curl

The application was configured with a local domain:

```text
http://whoami.localhost
```

Because the lab used SSH tunneling, the app was accessed from macOS using:

```bash
ssh -L 8081:localhost:80 ubuntu-machine
curl http://whoami.localhost:8081
```

Successful output included request metadata such as:

```text
Hostname: <container-id>
X-Forwarded-Host: whoami.localhost:8081
X-Forwarded-Server: <traefik-container-id>
```

## Lab 2: Docker Compose Deployment

The second deployment used Docker Compose to deploy another `whoami` service.

### Docker Compose File

```yaml
services:
  whoami:
    image: traefik/whoami:latest
    restart: unless-stopped
    networks:
      - coolify
    labels:
      - "traefik.enable=true"
      - "traefik.docker.network=coolify"
      - "traefik.http.routers.whoami-compose.rule=Host(`whoami-compose.localhost`)"
      - "traefik.http.routers.whoami-compose.entrypoints=http"
      - "traefik.http.services.whoami-compose.loadbalancer.server.port=80"

networks:
  coolify:
    external: true
```

### Purpose

- Understand how Coolify handles Docker Compose resources
- Learn how Traefik labels define routing
- Attach the application to the Coolify network
- Troubleshoot missing reverse proxy routes

Validation from Ubuntu:

```bash
curl -H "Host: whoami-compose.localhost" http://localhost
```

Validation from macOS:

```bash
ssh -L 8082:localhost:80 ubuntu-machine
curl http://whoami-compose.localhost:8082
```

Successful output included:

```text
Hostname: <container-id>
X-Forwarded-Host: whoami-compose.localhost:8082
X-Forwarded-Server: <traefik-container-id>
```

## Lab 3: PostgreSQL + Adminer

The third deployment used Docker Compose to deploy a PostgreSQL database with Adminer as a web UI.

### Docker Compose File

```yaml
services:
  postgres:
    image: postgres:16-alpine
    restart: unless-stopped
    environment:
      POSTGRES_DB: labdb
      POSTGRES_USER: labuser
      POSTGRES_PASSWORD: labpass123
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - coolify

  adminer:
    image: adminer:latest
    restart: unless-stopped
    environment:
      ADMINER_DEFAULT_SERVER: postgres
    networks:
      - coolify
    labels:
      - "traefik.enable=true"
      - "traefik.docker.network=coolify"
      - "traefik.http.routers.adminer-lab.rule=Host(`adminer.localhost`)"
      - "traefik.http.routers.adminer-lab.entrypoints=http"
      - "traefik.http.services.adminer-lab.loadbalancer.server.port=8080"

volumes:
  postgres_data:

networks:
  coolify:
    external: true
```

### Purpose

- Deploy a multi-container application stack
- Use environment variables for database initialization
- Connect a web application to PostgreSQL using Docker service discovery
- Validate persistent storage using Docker volumes
- Route Adminer through Traefik

Adminer was accessed from macOS using:

```bash
ssh -L 8083:localhost:80 ubuntu-machine
```

Browser URL:

```text
http://adminer.localhost:8083
```

Adminer login values:

```text
System: PostgreSQL
Server: postgres
Username: labuser
Password: labpass123
Database: labdb
```

## Database Validation

A test table was created:

```sql
CREATE TABLE notes (
    id SERIAL PRIMARY KEY,
    message TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

INSERT INTO notes (message)
VALUES ('Hello from Stevens Coolify lab + PostgreSQL lab');

SELECT * FROM notes;
```

The database was also validated from the Ubuntu terminal:

```bash
docker exec -it postgres-d1cmoet4067e3gqz0rmsy5cq psql -U labuser -d labdb -c "SELECT * FROM notes;"
```

Successful result:

```text
 id |                     message                     |         created_at         
----+-------------------------------------------------+----------------------------
  1 | Hello from Stevens Coolify lab + PostgreSQL lab | 2026-06-08 05:36:03.814131
(1 row)
```

## Persistence Test

The PostgreSQL container was restarted:

```bash
docker restart postgres-d1cmoet4067e3gqz0rmsy5cq
```

After restarting the container, the data was still available:

```bash
docker exec -it postgres-d1cmoet4067e3gqz0rmsy5cq psql -U labuser -d labdb -c "SELECT * FROM notes;"
```

Result:

```text
 id |                     message                     |         created_at         
----+-------------------------------------------------+----------------------------
  1 | Hello from Stevens Coolify lab + PostgreSQL lab | 2026-06-08 05:36:03.814131
(1 row)
```

This confirmed that the PostgreSQL data persisted through the Docker volume.

## Troubleshooting Notes

### Issue 1: VM IP changed after reboot

**Problem:**

The Ubuntu VM IP changed after reboot, making SSH access inconsistent.

**Fix:**

Configured a static IP using Netplan.

**Key lesson:**

For self-hosted labs, stable network access is important for SSH, port forwarding, and service testing.

---

### Issue 2: macOS could not access Coolify directly

**Problem:**

Coolify responded inside Ubuntu, but macOS could not directly reach the VM service URL because the VM was running behind VMware NAT.

**Fix:**

Used SSH port forwarding:

```bash
ssh -L 8000:localhost:8000 ubuntu-machine
```

**Key lesson:**

When running behind NAT, SSH tunneling can provide safe local access to internal services.

---

### Issue 3: Docker permission denied

**Problem:**

The user could not run Docker commands without sudo.

Error:

```text
permission denied while trying to connect to the docker API
```

**Fix:**

Added the user to the Docker group:

```bash
sudo usermod -aG docker $USER
```

Then logged out and back in.

**Key lesson:**

Docker socket access requires correct Linux group membership.

---

### Issue 4: Traefik returned 404 for Docker Compose app

**Problem:**

The Compose container was running, but accessing the domain returned:

```text
404 page not found
```

**Root cause:**

Traefik did not have a route for the hostname, and the service was not correctly attached to the Coolify network.

**Fix:**

Added Traefik labels and attached the service to the external `coolify` network.

**Key lesson:**

A running container is not enough. For a web app to be reachable through Coolify, the reverse proxy must have a valid route and network path to the container.

---

### Issue 5: Wrong place to test local tunnel

**Problem:**

Tried to test `localhost:8082` from inside Ubuntu.

**Fix:**

The SSH tunnel port exists on macOS, not Ubuntu.

Correct test from macOS:

```bash
curl http://whoami-compose.localhost:8082
```

**Key lesson:**

SSH local port forwarding creates the listening port on the client machine, not on the remote server.

## Useful Commands

```bash
docker ps
docker ps -a
docker compose ls
docker network ls
docker logs <container>
docker inspect <container>
curl -H "Host: hostname.localhost" http://localhost
ssh -L local_port:localhost:remote_port ubuntu-machine
```

## Key Skills Practiced

- Coolify installation
- Self-hosted PaaS operations
- Docker image deployment
- Docker Compose deployment
- Traefik reverse proxy routing
- Host-based routing
- Docker networking
- SSH port forwarding
- PostgreSQL deployment
- Environment variables
- Docker volumes and persistence
- Linux troubleshooting
- Validation using `curl`, `docker ps`, `docker inspect`, `docker compose ls`, and `psql`

## Outcome

Successfully installed Coolify on an Ubuntu VM, deployed applications using both Docker Image and Docker Compose workflows, configured Traefik routing, deployed PostgreSQL with Adminer, and confirmed persistent database storage after a container restart.
