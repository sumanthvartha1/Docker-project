# Mini GPay

A payment app I built to learn Docker, CI/CD, and Kubernetes from scratch. The app lets users check balances, send money, and view transaction history — nothing fancy, but the infrastructure behind it covers real production patterns.

## What I built

Five containers working together — a Flask API backend, PostgreSQL database, Redis cache, static HTML frontend, and nginx as a reverse proxy. Everything orchestrated first with Docker Compose locally, then deployed through a Jenkins CI/CD pipeline to AWS.

```
browser → nginx (port 80) → routes /api to backend, / to frontend
                              backend → postgres (stores data)
                              backend → redis (caches balances)
```

## Architecture

<img width="1442" height="1000" alt="image" src="https://github.com/user-attachments/assets/368df285-ada3-48a1-9c46-e972e699fefc" />



Two separate Docker networks. The database sits on the backend network only — nginx and the frontend can never reach it directly. The backend bridges both networks. This is the same isolation pattern banks use in production.

## Tech stack

- **Backend:** Python, Flask, psycopg2, redis-py
- **Database:** PostgreSQL 15 (Alpine)
- **Cache:** Redis 7 (Alpine)
- **Frontend:** HTML, CSS, vanilla JavaScript
- **Proxy:** Nginx (Alpine)
- **Containers:** Docker, Docker Compose

## Project structure

```
mini-gpay/
├── docker-compose.yml        # local development setup
├── backend/
│   ├── Dockerfile
│   ├── app.py                # Flask API — 5 endpoints
│   └── requirements.txt
├── frontend/
│   ├── Dockerfile
│   └── index.html            # payment UI
├── nginx/
│   ├── Dockerfile
│   └── nginx.conf            # routing rules
├── postgres-init/
   └── init.sql              # creates tables + sample data

```

## How to run locally

```bash
git clone https://github.com/sumanthvartha1/mini-gpay-cicd.git
cd mini-gpay-cicd
docker compose up --build
```

Open `http://localhost:80`. Select a user, send money, check history.

Stop: `docker compose down`

## API endpoints

```
GET  /api/users                → list all users
GET  /api/balance/:id          → get balance (checks Redis first, falls back to Postgres)
POST /api/send                 → transfer money (body: sender_id, receiver_id, amount)
GET  /api/transactions/:id     → transaction history for a user
GET  /api/health               → health check
```

Send money example:
```json
POST /api/send
{
    "sender_id": 1,
    "receiver_id": 2,
    "amount": 500
}

Things I Learned Building This

Docker networking — containers are isolated by default. Putting them on the same network lets them find each other by service name through Docker's built-in DNS. IP addresses change on restart, names don't.

Layer caching matters — copying requirements.txt before the code means pip install gets cached. Changed one line in app.py? Build goes from 3 minutes to 8 seconds. Across 20 deploys a day that adds up fast.

Volumes vs bind mounts — named volumes for database persistence (Docker manages the storage), bind mounts for development (edit code locally, container sees changes instantly).

Health checks prevent race conditions — without them, the backend starts before PostgreSQL is ready and crashes with "connection refused." With condition: service_healthy, the backend waits until pg_isready passes.

Network isolation is real security — two separate networks means even if nginx gets compromised, the attacker can't reach the database. The backend is the only bridge between networks.


Known Limitations

This is a learning project. Production gaps include:


No TLS — HTTP only, no HTTPS/SSL termination
Hardcoded secrets — Passwords in plain text in docker-compose.yml (use Docker secrets or external vault in production)
No resource limits — No CPU/memory constraints on containers
No Redis auth — Redis accepts connections without a password
No connection pooling — Backend opens a new database connection per request
No rate limiting — API endpoints have no request throttling
Missing restart policies — Postgres, Redis, and Frontend lack restart: unless-stopped
Single-host only — Bridge networking doesn't span multiple hosts (use Docker Swarm or Kubernetes for multi-host)



Built With

This project was built as a hands-on Docker Compose exercise covering:


Multi-container orchestration with Docker Compose
User-defined bridge networks for container isolation
Health checks with dependency ordering (depends_on + condition: service_healthy)
Named volumes for persistent database storage
Reverse proxy pattern with Nginx
Cache-aside pattern with Redis                     
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        
