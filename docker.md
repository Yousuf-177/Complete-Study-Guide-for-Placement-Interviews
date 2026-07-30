# Docker Interview Cheat Sheet — Software Engineering Placements

*A senior-DevOps-interviewer-style prep guide: concise, technically complete answers with code snippets.*

---

## Table of Contents
1. [Core Concepts](#core-concepts)
2. [Architecture](#architecture)
3. [Images & Dockerfile](#images--dockerfile)
4. [Commands](#commands)
5. [Networking](#networking)
6. [Volumes & Storage](#volumes--storage)
7. [Docker Compose](#docker-compose)
8. [Security](#security)
9. [Real-World Scenarios](#real-world-scenarios)
10. [Rapid-Fire Round](#rapid-fire-round)

---

## Core Concepts

### 1. What is Docker, and why is it used?
Docker is a platform for building, shipping, and running applications inside **containers** — lightweight, isolated units that bundle an app with its code, runtime, libraries, and config. It solves the "works on my machine" problem by making environments reproducible across dev, test, and prod.

### 2. What is a container? How is it different from a Virtual Machine?
A container is an isolated process that shares the host OS kernel but has its own filesystem, network, and process namespace. A VM virtualizes an entire OS (including its own kernel) on top of a hypervisor.

| Aspect | Container | VM |
|---|---|---|
| Boot time | Seconds | Minutes |
| Size | MBs | GBs |
| Kernel | Shared with host | Own kernel |
| Isolation | Process-level (namespaces/cgroups) | Full hardware-level |
| Density | Many per host | Few per host |

**Why it's asked:** Tests whether you understand *why* containers are lightweight — the answer must mention kernel sharing, not just "containers are smaller."

### 3. What is a Docker image?
A read-only template made of stacked, immutable **layers**, containing the application, its dependencies, and metadata (entrypoint, env vars, exposed ports). Running an image creates a container — a writable instance of it.

### 4. What is the difference between an image and a container?
An image is the blueprint (static); a container is a running instance of that blueprint (a process + a thin writable layer on top of the image layers). You can spin up many containers from one image.

### 5. What is Docker Hub?
A public (and private) cloud registry for storing and distributing Docker images — the default source for `docker pull`/`docker push`, similar to how GitHub hosts code.

### 6. What are namespaces and cgroups, and how does Docker use them?
- **Namespaces** (PID, NET, MNT, UTS, IPC, USER) give a container its own isolated view of processes, network, mounts, hostname, etc. — this is what makes a container *look* like a separate machine.
- **cgroups** (control groups) limit and account for resource usage (CPU, memory, I/O) per container.

Together they're the Linux kernel primitives Docker builds on — Docker itself doesn't invent isolation, it orchestrates these two features.

### 7. What is a Dockerfile?
A plain-text script of instructions (`FROM`, `RUN`, `COPY`, `CMD`, etc.) that Docker reads top-to-bottom to build an image, layer by layer.

---

## Architecture

### 8. Explain Docker's client-server architecture.
- **Docker Client** — the CLI you type commands into (`docker run`, `docker build`).
- **Docker Daemon (dockerd)** — a background process that does the real work: building images, running containers, managing networks/volumes. Exposes a REST API.
- **Docker Registry** — stores images (e.g., Docker Hub, ECR, private registries).

```
docker CLI  --REST API-->  dockerd (daemon)  --pulls/pushes-->  Registry
                                |
                                v
                        containerd -> runc -> Linux kernel (namespaces/cgroups)
```

### 9. What is containerd and runc? Where do they fit?
- **containerd** is a high-level container runtime (manages image transfer, storage, container lifecycle) that `dockerd` delegates to.
- **runc** is the low-level OCI-compliant runtime that actually creates the container process using namespaces/cgroups.

Docker Engine = dockerd + containerd + runc, following the **OCI (Open Container Initiative)** spec so images/runtimes are interoperable across tools (Docker, Podman, Kubernetes CRI).

### 10. What is Docker Engine?
The core software that builds and runs containers — the combination of the daemon, REST API, and CLI.

### 11. How does Docker achieve image layering, and why does it matter?
Each Dockerfile instruction that changes the filesystem creates a new **read-only layer**, cached and content-addressed by hash. Layers are stacked using a **union filesystem** (commonly overlay2). This matters because:
- Unchanged layers are cached and reused across builds (faster builds).
- Multiple images sharing base layers only store that layer once on disk (space savings).

### 12. What is a union filesystem / OverlayFS?
A filesystem that transparently overlays multiple directories (layers) into a single unified view. Docker's default storage driver, `overlay2`, uses this to stack read-only image layers under one writable container layer.

---

## Images & Dockerfile

### 13. Write a minimal Dockerfile for a Node.js app.
```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

### 14. What's the difference between `CMD` and `ENTRYPOINT`?
- `ENTRYPOINT` defines the fixed executable that always runs.
- `CMD` supplies default arguments, which can be overridden at `docker run`.
- Best practice: combine them — `ENTRYPOINT ["python", "app.py"]` + `CMD ["--env=prod"]` — so the base command is locked but flags are overridable.

### 15. What's the difference between `COPY` and `ADD`?
`COPY` copies local files/directories as-is. `ADD` does the same but also auto-extracts local tar archives and can fetch remote URLs. Best practice: prefer `COPY` unless you specifically need `ADD`'s extra behavior — it's more predictable.

### 16. What is a multi-stage build, and why use one?
It lets you use multiple `FROM` stages in one Dockerfile so build tools/dependencies stay in an early stage and only the final artifact is copied into a slim runtime image — dramatically shrinking final image size.

```dockerfile
# Stage 1: build
FROM golang:1.22 AS builder
WORKDIR /src
COPY . .
RUN go build -o app .

# Stage 2: run
FROM alpine:3.20
COPY --from=builder /src/app /usr/local/bin/app
ENTRYPOINT ["app"]
```

**Why it's asked:** Directly tests real-world image-optimization skill, not just syntax memorization.

### 17. How do you reduce Docker image size?
- Use minimal base images (`alpine`, `distroless`, `slim` variants).
- Use multi-stage builds to drop build-time dependencies.
- Combine `RUN` commands to reduce layer count and clean up caches in the same layer (e.g., `apt-get install && rm -rf /var/lib/apt/lists/*` in one `RUN`).
- Add a `.dockerignore` to avoid copying unnecessary files (`.git`, `node_modules`, logs).
- Avoid installing unnecessary packages/dev tools in the final image.

### 18. What is a `.dockerignore` file for?
Like `.gitignore` — it excludes files/folders (e.g., `.git`, `node_modules`, `*.log`) from the build context sent to the daemon, speeding up builds and avoiding accidental leaks of secrets or bloat.

### 19. What does "dangling image" mean?
An image layer left with `<none>` as its tag — usually the old version of an image after a rebuild overwrote its tag. Clean up with:
```bash
docker image prune
```

### 20. How does Docker layer caching affect build order in a Dockerfile?
Docker caches each layer and reuses it if the instruction and its inputs haven't changed. So you should order instructions from least-to-most frequently changing — e.g., copy `package.json` and install dependencies *before* copying the full source code, so code changes don't invalidate the (slow) dependency-install layer.

---

## Commands

### 21. Core commands cheat sheet

```bash
# Images
docker build -t myapp:1.0 .          # build an image from Dockerfile
docker images                        # list local images
docker rmi myapp:1.0                 # remove an image
docker pull nginx:latest             # pull from registry
docker push myrepo/myapp:1.0         # push to registry

# Containers
docker run -d -p 8080:80 --name web nginx   # run detached, map ports
docker ps                            # list running containers
docker ps -a                         # list all containers (incl. stopped)
docker stop web && docker start web  # stop / start
docker rm web                        # remove a stopped container
docker exec -it web bash             # shell into a running container
docker logs -f web                   # stream logs

# Inspect / debug
docker inspect web                   # full metadata as JSON
docker stats                         # live resource usage
docker top web                       # processes inside container

# Cleanup
docker system prune -a               # remove unused images/containers/networks
```

### 22. What's the difference between `docker run` and `docker start`?
`docker run` creates a **new** container from an image (and starts it). `docker start` restarts an **existing, stopped** container by its ID/name.

### 23. What does `docker exec -it <container> bash` do?
Opens an interactive (`-i`) TTY (`-t`) shell session inside an already-running container — the standard way to debug live containers.

### 24. What's the difference between `docker stop` and `docker kill`?
`docker stop` sends `SIGTERM`, waits a grace period (default 10s) for graceful shutdown, then sends `SIGKILL` if needed. `docker kill` sends `SIGKILL` (or a specified signal) immediately — no grace period.

### 25. How do you check resource usage of running containers?
```bash
docker stats
```
Shows live CPU %, memory usage/limit, network I/O, and block I/O per container.

---

## Networking

### 26. What are the default Docker network drivers?
- **bridge** (default) — private internal network on a single host; containers talk to each other via container name as DNS.
- **host** — container shares the host's network namespace directly (no isolation, no port mapping needed).
- **overlay** — connects containers across multiple Docker hosts (used in Swarm/multi-host setups).
- **none** — no networking at all.

### 27. How do two containers communicate with each other?
Put them on the same **user-defined bridge network** — Docker's embedded DNS then lets them resolve each other by container name.
```bash
docker network create app-net
docker run -d --name db --network app-net postgres
docker run -d --name api --network app-net myapi
# inside 'api' container: connect to host "db" on its port
```

### 28. What does `-p 8080:80` mean in `docker run`?
Maps **host port 8080** to **container port 80** (`-p host:container`), so external traffic hitting the host on 8080 reaches the app listening on 80 inside the container.

### 29. Why is a user-defined bridge network preferred over the default `bridge`?
The default bridge network doesn't provide automatic DNS resolution by container name (you'd need `--link`, which is deprecated). A user-defined bridge gives automatic service discovery, better isolation, and easier reconfiguration.

---

## Volumes & Storage

### 30. How does Docker handle persistent data? Volumes vs. bind mounts.
- **Volumes** — managed by Docker, stored under `/var/lib/docker/volumes/`, portable, backup-friendly, decoupled from host filesystem structure. Preferred for production.
- **Bind mounts** — map a specific host path directly into the container. Useful for local development (e.g., live-reloading source code), but tightly coupled to host layout.

```bash
# Named volume
docker volume create app-data
docker run -v app-data:/var/lib/mysql mysql

# Bind mount
docker run -v $(pwd)/src:/app/src myapp
```

### 31. Why are containers considered "ephemeral," and how do you avoid losing data?
A container's writable layer is deleted when the container is removed — any data written only there is lost. Attach a **volume** for anything that must persist (databases, uploads) so data survives container restarts/removal.

### 32. What is a "tmpfs mount" and when would you use it?
A mount stored in host memory (RAM) only, never written to disk — useful for sensitive, temporary data (e.g., secrets processed at runtime) since it disappears when the container stops.

---

## Docker Compose

### 33. What is Docker Compose, and why use it?
A tool to define and run multi-container applications using a single declarative YAML file, replacing long chains of manual `docker run` commands. One command spins the whole stack up.

### 34. Write a `docker-compose.yml` for a web app + database.
```yaml
version: "3.9"
services:
  web:
    build: .
    ports:
      - "3000:3000"
    depends_on:
      - db
    environment:
      - DATABASE_URL=postgres://user:pass@db:5432/appdb
  db:
    image: postgres:16
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
      - POSTGRES_DB=appdb
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:
```
```bash
docker compose up -d      # start everything in the background
docker compose down       # stop and remove containers/networks
docker compose logs -f web
```

### 35. What does `depends_on` actually guarantee (and not guarantee)?
It controls **startup order only** — it waits for the dependency container to *start*, not for the service inside it to be *ready* (e.g., Postgres accepting connections). For real readiness, use healthchecks or an app-level retry/wait mechanism.

### 36. How is Docker Compose different from Docker Swarm/Kubernetes?
Compose is for defining and running multi-container apps on a **single host** (great for dev/test). Swarm and Kubernetes are **orchestrators** for running and scaling containers across a **cluster of machines**, adding scheduling, self-healing, load balancing, and rolling updates.

---

## Security

### 37. How do you avoid running a container as root?
Add a non-root user in the Dockerfile:
```dockerfile
RUN addgroup -S app && adduser -S app -G app
USER app
```
Running as root inside a container is a common audit finding — if an attacker escapes the container, they'd have root on a shared kernel.

### 38. How do you manage secrets in Docker — and what should you avoid?
**Avoid:** baking secrets into image layers via `ENV` or `ARG` (they persist in image history and `docker history`/`inspect`).
**Prefer:** Docker Secrets (in Swarm mode), mounted secret files, external secret managers (Vault, AWS Secrets Manager), or `--env-file` supplied only at runtime (not baked into the image).

### 39. How would you scan a Docker image for vulnerabilities?
```bash
docker scout cves myimage:1.0
# or
trivy image myimage:1.0
```
Both inspect installed packages/layers against CVE databases — a standard CI/CD gate before pushing images to production registries.

### 40. What is image signing, and why does it matter?
Cryptographically signing images (e.g., with Docker Content Trust / Notary, or Sigstore/cosign) so consumers can verify an image genuinely came from a trusted publisher and wasn't tampered with in the registry or in transit.

---

## Real-World Scenarios

### 41. Your app works locally but crashes right after deploying to production. How do you debug it?
1. `docker logs <container>` — check the crash reason/stack trace.
2. `docker inspect <container>` — check exit code, env vars, mounted volumes.
3. Confirm parity: same image tag used in both environments, same env vars/config, no reliance on host-only paths.
4. Check resource limits — production may enforce memory/CPU caps that trigger OOM kills locally you didn't hit.

### 42. Your production container keeps getting OOM-killed. How do you investigate and fix it?
1. `docker stats` / `docker inspect` to confirm memory limit vs. actual usage.
2. Check exit code `137` (`docker ps -a`), which indicates a SIGKILL from an OOM condition.
3. Profile the app for memory leaks; tune the app's own memory settings (e.g., JVM heap size) to fit inside the container limit.
4. Adjust the container's memory limit (`--memory`) if the workload genuinely needs more, or add a memory-based autoscaling policy.

### 43. Your CI/CD pipeline is slow because Docker image builds take too long. How do you speed it up?
- Reorder Dockerfile instructions so rarely-changing layers (dependency installs) come before frequently-changing ones (source code).
- Use build caching (`--cache-from`, BuildKit's cache mounts/registry cache).
- Use multi-stage builds to avoid rebuilding heavy toolchains repeatedly.
- Parallelize independent build stages with BuildKit.
```bash
DOCKER_BUILDKIT=1 docker build --cache-from myrepo/myapp:cache -t myapp:latest .
```

### 44. You need zero-downtime deployments for a customer-facing app running in containers. How do you implement it?
Use a **rolling update** strategy: bring up new containers behind a load balancer, wait for health checks to pass, then drain and remove old containers one at a time — never taking all instances down simultaneously. Orchestrators (Kubernetes, Swarm, ECS) provide this natively via rolling update configs and readiness probes.

### 45. A security audit flags multiple vulnerabilities in your images. What's your remediation plan?
1. Re-base onto a smaller, actively patched base image (e.g., distroless or updated Alpine).
2. Run `docker scout` / Trivy in CI to block builds above a severity threshold.
3. Patch/upgrade flagged packages, remove unused ones.
4. Automate periodic rebuilds so base-image CVE patches flow through even without app code changes.

### 46. Your team is migrating a monolithic app into Docker containers. How do you approach it?
1. **Containerize as-is first** ("lift and shift") to establish a baseline and unblock CI/CD — don't try to decompose and containerize simultaneously.
2. Externalize config via environment variables (12-factor app principles).
3. Identify natural service boundaries for later decomposition (don't force microservices on day one).
4. Add health checks, structured logging (stdout/stderr), and graceful shutdown handling so the app plays well with orchestrators.

### 47. Multiple containers need to communicate securely while staying isolated from other services. How do you design the network?
Use separate **user-defined bridge/overlay networks** per logical boundary (e.g., `frontend-net`, `backend-net`) and attach each container only to the networks it needs — a database container should not sit on the same network as the public-facing web tier. Use network policies (in Kubernetes) or Swarm's overlay encryption for cross-host traffic.

### 48. How would you set up a health check so orchestration tools know if your container is actually ready?
```dockerfile
HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1
```
Orchestrators use this status (`healthy`/`unhealthy`) to decide whether to route traffic to the container or restart it.

---

## Rapid-Fire Round
*(Quick one-liners interviewers often close with.)*

| Question | Answer |
|---|---|
| Default storage driver? | `overlay2` |
| Command to view container processes? | `docker top <container>` |
| Command to see build layer history? | `docker history <image>` |
| How to limit container memory? | `docker run --memory=512m ...` |
| How to limit CPU? | `docker run --cpus=1.5 ...` |
| File to exclude build context items? | `.dockerignore` |
| Restart policy for always-on services? | `--restart unless-stopped` or `always` |
| Command to see disk usage by Docker? | `docker system df` |
| How to tag an image? | `docker tag myapp:1.0 myrepo/myapp:1.0` |
| OCI stands for? | Open Container Initiative |

---

### Prep Tip
- **Freshers:** Focus on *Core Concepts*, *Commands*, and *Images & Dockerfile* — interviewers weight these heavily for entry-level roles.
- **1–3 YOE / mid-level:** Add *Networking*, *Volumes*, and *Docker Compose* — expect at least one hands-on Compose or Dockerfile-writing exercise.
- **Senior/DevOps-focused roles:** Be ready to go deep on *Security* and *Real-World Scenarios* — these separate candidates who've only used Docker locally from those who've run it in production.
