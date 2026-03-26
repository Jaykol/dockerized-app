![CI/CD Pipeline](https://github.com/Jaykol/dockerized-app/actions/workflows/docker-build.yml/badge.svg)
[![Docker Build](https://github.com/Jaykol/dockerized-app/actions/workflows/docker-build.yml/badge.svg)](https://github.com/Jaykol/dockerized-app/actions/workflows/docker-build.yml)

# Dockerized Three-Tier App

A production-ready containerized web application built with Flask, Nginx, and PostgreSQL — orchestrated with Docker Compose.

## Architecture
```
Browser → Nginx (port 80) → Flask/Gunicorn (port 5000) → PostgreSQL (port 5432)
```

- **Nginx** — reverse proxy, handles incoming HTTP traffic
- **Flask + Gunicorn** — REST API application server
- **PostgreSQL** — persistent relational database

## Project Structure
```
dockerized-app/
├── app/
│   ├── app.py              # Flask application
│   ├── requirements.txt    # Python dependencies
│   └── Dockerfile          # App container definition
├── nginx/
│   ├── nginx.conf          # Reverse proxy configuration
│   └── Dockerfile          # Nginx container definition
├── docker-compose.yml      # Multi-container orchestration
├── .env.example            # Required environment variables
└── .gitignore
```

## Quick Start

**1. Clone the repo:**
```bash
git clone https://github.com/Jaykol/dockerized-app.git
cd dockerized-app
```

**2. Set up environment variables:**
```bash
cp .env.example .env
# Edit .env and set a strong DB_PASSWORD
```

**3. Build and run:**
```bash
docker compose up --build
```

**4. Test the API:**
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

## API Endpoints

| Method | Endpoint  | Description       |
|--------|-----------|-------------------|
| GET    | /health   | Health check      |
| GET    | /tasks    | List all tasks    |
| POST   | /tasks    | Create a new task |

## Key Features

- Multi-stage Docker networking with named services
- PostgreSQL data persistence via Docker volumes
- Database healthcheck — app waits for db to be ready
- Credentials managed via environment variables — never hardcoded
- Gunicorn production WSGI server
- Nginx reverse proxy with upstream configuration

## Environment Variables

| Variable    | Description              |
|-------------|--------------------------|
| DB_PASSWORD | PostgreSQL user password |

## Author

Jesutofunmi Ajekola — [GitHub](https://github.com/Jaykol) | [LinkedIn](https://www.linkedin.com/in/jesutofunmij)
# CI/CD Pipeline: test -> build -> push -> deploy

## CI/CD Pipeline

Every push to `main` triggers a three-stage automated pipeline:
```
test → build-and-push → deploy
```

- **test** — spins up the full stack and runs API endpoint tests
- **build-and-push** — builds the Docker image and pushes to Docker Hub with two tags: `latest` and the commit SHA
- **deploy** — SSHs into EC2, pulls the new image, and restarts the container

### Pipeline triggers
- `push` to `main` — runs all three jobs
- `pull_request` to `main` — runs `test` only (no deployment)

## Infrastructure

- **EC2** — Amazon Linux 2023, t2.micro, runs Flask + Nginx containers
- **RDS** — PostgreSQL 15, db.t3.micro, private subnet (no public access)
- **Docker Hub** — image registry at `jaykol/dockerized-app`
