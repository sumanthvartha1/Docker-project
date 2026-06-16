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

                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        
