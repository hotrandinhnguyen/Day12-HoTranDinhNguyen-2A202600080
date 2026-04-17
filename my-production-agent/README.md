# Lab 12 - Complete Production Agent

Project tong hop cac yeu cau production cho Day 12: auth, rate limit, cost guard, Docker, Redis, readiness check, va deploy.

## Checklist Deliverable

- [x] Dockerfile multi-stage
- [x] `docker-compose.yml` cho agent + Redis + nginx
- [x] `.dockerignore`
- [x] `GET /health`
- [x] `GET /ready`
- [x] API key authentication
- [x] Rate limiting `10 req/min`
- [x] Cost guard `$10/month`
- [x] Config tu environment variables
- [x] Structured logging
- [x] Graceful shutdown
- [x] Railway / Render config

## Structure

```text
my-production-agent/
|-- app/
|   |-- main.py
|   |-- config.py
|   |-- auth.py
|   |-- rate_limiter.py
|   `-- cost_guard.py
|-- utils/
|   `-- mock_llm.py
|-- Dockerfile
|-- docker-compose.yml
|-- nginx.conf
|-- railway.toml
|-- render.yaml
|-- .env.example
|-- .dockerignore
|-- requirements.txt
`-- check_production_ready.py
```

## Run Local

Run all commands below inside `my-production-agent/`.

### 1. Prepare local environment

Create `.env.local` from the template:

```bash
cp .env.example .env.local
```

If you are using PowerShell, this also works:

```powershell
Copy-Item .env.example .env.local
```

### 2. Start the full stack

```bash
docker compose up --build
```

This starts:
- `agent`
- `redis`
- `nginx`

The app is exposed through nginx at `http://localhost`.

### 3. Health check

```bash
curl http://localhost/health
curl http://localhost/ready
```

### 4. Authenticated request

Use the API key value from `.env.local` (`AGENT_API_KEY=...`):

```bash
curl -X POST http://localhost/ask \
  -H "X-API-Key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"user_id":"demo","question":"What is deployment?"}'
```

### 5. Production readiness self-check

```bash
python check_production_ready.py
```

## Deploy

### Railway

```bash
railway login
railway init
railway variables set ENVIRONMENT=production
railway variables set AGENT_API_KEY=your-secret-key
railway variables set JWT_SECRET=your-jwt-secret
railway variables set MONTHLY_BUDGET_USD=10
railway up
```

Notes:
- `ENVIRONMENT=production` is required for production mode
- `OPENAI_API_KEY` is optional in this lab because the project can run with the mock LLM
- `REDIS_URL` is usually provided by the Railway Redis service

### Render

1. Push the repo to GitHub.
2. Create a service from `render.yaml`.
3. Set secrets `ENVIRONMENT`, `AGENT_API_KEY`, `JWT_SECRET`, and `MONTHLY_BUDGET_USD`.
4. Deploy and test `GET /health`, `GET /ready`, and `POST /ask`.

## Public Deployment Used For Submission

- URL: `https://day12-final-production.up.railway.app`
- Platform: Railway

Quick verification:

```bash
curl https://day12-final-production.up.railway.app/health
curl https://day12-final-production.up.railway.app/ready
```

## Notes For Grading

- Final application source is in `my-production-agent/`
- Submission documents are in the repo root:
  - `MISSION_ANSWERS.md`
  - `DEPLOYMENT.md`
  - `screenshots/`
