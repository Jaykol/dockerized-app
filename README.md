![CI/CD Pipeline](https://github.com/Jaykol/dockerized-app/actions/workflows/docker-build.yml/badge.svg)
[![Docker Build](https://github.com/Jaykol/dockerized-app/actions/workflows/docker-build.yml/badge.svg)](https://github.com/Jaykol/dockerized-app/actions/workflows/docker-build.yml)

# Dockerized Three-Tier App with Secure CI/CD

A production-ready containerized web application built with Flask, Nginx, and PostgreSQL — 
secured with a DevSecOps pipeline featuring automated vulnerability scanning, secret detection, 
SAST, and dependency auditing on every commit.

![CI/CD](https://github.com/Jaykol/dockerized-app/actions/workflows/docker-build.yml/badge.svg)

---

## Architecture
```
Browser → Nginx (port 80) → Flask/Gunicorn (port 5000) → PostgreSQL (port 5432)
```

| Component | Role |
|---|---|
| Nginx | Reverse proxy, handles incoming HTTP traffic |
| Flask + Gunicorn | REST API application server |
| PostgreSQL | Persistent relational database |

---

## Secure CI/CD Pipeline

Every push to `main` runs a **7-stage security-gated pipeline**. A failure at any stage blocks deployment.
test → secret-scan → sast → dependency-scan → build-and-push → image-scan → deploy

| Stage | Tool | What It Catches |
|---|---|---|
| `test` | pytest | Regression failures |
| `secret-scan` | Gitleaks | Hardcoded credentials, API keys in code |
| `sast` | Semgrep | Insecure code patterns (SQLi, XSS, unsafe deserialization) |
| `dependency-scan` | Snyk | CVEs in Python dependencies (requirements.txt) |
| `build-and-push` | Docker Buildx | Builds and pushes image to Docker Hub |
| `image-scan` | Trivy | OS and library CVEs in the container image |
| `deploy` | SSH | Deploys to EC2 only if all security gates pass |

### Security findings are uploaded to the GitHub Security tab (SARIF format)

### Pipeline triggers
- `push` to `main` — full 7-stage pipeline including deployment
- `pull_request` to `main` — security gates run, no deployment

### Security decisions documented
- Base image migrated from `python:3.11-slim` (Debian) to `python:3.11-alpine` to reduce attack surface
- Upgraded `gunicorn` to 22.0.0 (CVE-2024-1135, CVE-2024-6827 — HTTP request smuggling)
- Upgraded `wheel` to 0.46.2 (CVE-2026-24049 — privilege escalation)
- Upgraded `setuptools` to 80.0.0
- `apk upgrade` in Dockerfile to patch `zlib` CVE-2026-22184
- Vendor-internal CVEs in setuptools-bundled packages documented and suppressed in `.trivyignore`

---

## Project Structure


```
dockerized-app/
├── app/
│   ├── app.py              # Flask application
│   ├── requirements.txt    # Python dependencies
│   └── Dockerfile          # App container (python:3.11-alpine)
├── nginx/
│   ├── nginx.conf          # Reverse proxy configuration
│   └── Dockerfile          # Nginx container
├── tests/
│   └── test_app.py         # Pytest test suite
├── .github/
│   └── workflows/
│       └── docker-build.yml  # 7-stage secure CI/CD pipeline
├── .trivyignore            # Documented CVE suppressions
├── .gitleaks.toml          # Gitleaks configuration
├── docker-compose.yml      # Local multi-container orchestration
├── .env.example            # Required environment variables
└── .gitignore
```

---

## Infrastructure

| Resource | Details |
|---|---|
| EC2 | Amazon Linux 2023, ARM64 t4g.micro — runs Flask + Nginx containers |
| RDS | PostgreSQL 15, db.t3.micro, private subnet (no public access) |
| Security Groups | EC2 SG scoped to ports 22/80/443/5000; RDS SG allows only EC2 SG |
| Docker Hub | Image registry — `jaykol/taskapp:latest` + `jaykol/taskapp:<sha>` |

---

## Quick Start (Local)

1. Clone the repo:
```bash
git clone https://github.com/Jaykol/dockerized-app.git
cd dockerized-app
```

2. Set up environment variables:
```bash
cp .env.example .env
# Edit .env and set a strong DB_PASSWORD
```

3. Build and run:
```bash
docker compose up --build
```

4. Test the API:
```bash
# Health check
curl http://localhost/health

# Create a task
curl -X POST http://localhost/tasks \
  -H "Content-Type: application/json" \
  -d '{"title": "My first task"}'

# Get all tasks
curl http://localhost/tasks
```

---

## API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| GET | `/health` | Health check |
| GET | `/tasks` | List all tasks |
| POST | `/tasks` | Create a new task |

---

## GitHub Secrets Required

| Secret | Description |
|---|---|
| `DOCKERHUB_USERNAME` | Docker Hub username |
| `DOCKERHUB_TOKEN` | Docker Hub access token |
| `EC2_HOST` | EC2 public IP address |
| `EC2_SSH_KEY` | Private SSH key for EC2 access |
| `DATABASE_URL` | PostgreSQL connection string |
| `SNYK_TOKEN` | Snyk API token for dependency scanning |
| `SEMGREP_APP_TOKEN` | Semgrep token for SAST |

---

## Author

Jesutofunmi Ajekola — [GitHub](https://github.com/Jaykol) | [LinkedIn](https://www.linkedin.com/in/jesutofunmij)
