# AI Recruitment Agent

A runnable, security-conscious starter project for resume screening, candidate shortlisting, interview scheduling, and email reminders.

## What is included
- FastAPI REST API with Swagger UI at `/docs`
- JWT login and role-aware endpoints (`admin`, `recruiter`)
- PostgreSQL in Docker Compose; SQLite can be used for quick local testing
- Resume upload and text extraction for PDF and DOCX
- Ollama-powered resume analysis with a transparent skill score + TF-IDF similarity component
- Recruiter-controlled shortlist status (AI score is advisory, not an automatic hiring decision)
- Interview scheduling and reminder worker
- Email delivery through SMTP, with safe console logging when SMTP is not configured
- Dockerfile, tests, environment template, and a minimal dashboard

## Important
This is a production-oriented **starter**, not a certified or fully audited production deployment. Ollama must be installed and its model pulled separately; it is not bundled into the ZIP. Before real candidate data is used, complete the security/compliance checklist below, conduct penetration testing, and have HR/legal review your hiring workflow. Do not use protected characteristics or proxies to rank candidates. Keep a human reviewer in the loop.

## 1. Requirements
- Docker Desktop + Docker Compose (recommended), or Python 3.11+
- Optional SMTP credentials for real email delivery

## Ollama setup (local LLM)
Install Ollama from https://ollama.com/ and pull a model:
```bash
ollama pull qwen2.5:7b
ollama list
```
For Docker Desktop, `.env.example` uses `OLLAMA_BASE_URL=http://host.docker.internal:11434` so the API container can reach Ollama running on your host. If running the API directly on your host, use `http://localhost:11434`.
For smaller RAM systems, choose a smaller model such as `qwen2.5:3b` and update `OLLAMA_MODEL`. Model quality and speed depend on your hardware.

## 2. Start with Docker
```bash
cp .env.example .env
docker compose up --build
```
Open:
- Dashboard: http://localhost:8000/
- API docs: http://localhost:8000/docs
- Health: http://localhost:8000/health

First admin is created from `BOOTSTRAP_ADMIN_EMAIL` and `BOOTSTRAP_ADMIN_PASSWORD` in `.env`.
Change these before running anywhere beyond local development.

## 3. Run locally without Docker
```bash
python -m venv .venv
# Windows:
.venv\Scripts\activate
# macOS/Linux:
source .venv/bin/activate
pip install -r requirements.txt
copy .env.example .env   # Windows; use cp on macOS/Linux
uvicorn app.main:app --reload
```
For SQLite local mode, set:
`DATABASE_URL=sqlite:///./recruitment.db`

## 4. Typical workflow
1. Sign in via `POST /auth/login` using form fields `username` (email) and `password`.
2. Create a job: `POST /jobs`
3. Upload a candidate resume: `POST /candidates` as multipart form fields `full_name`, `email`, `resume`.
4. Match candidates against a job: `POST /jobs/{job_id}/match`
5. Review score + reasons: `GET /jobs/{job_id}/matches`
6. A recruiter can shortlist/reject a candidate explicitly: `PATCH /applications/{application_id}`
7. Schedule an interview: `POST /interviews`
8. Reminders are checked periodically. If SMTP is configured, email is sent; otherwise the message is logged.

## API notes
See `/docs` for schemas and interactive testing. All dates must be ISO-8601 with timezone, for example `2026-10-20T10:30:00+05:30`.

## Environment variables
See `.env.example`. Use a strong unique `SECRET_KEY`. Set `CORS_ORIGINS` to your real frontend origin(s), not `*`, in production.

## Production hardening checklist
- Use a managed PostgreSQL service with encrypted storage and backups.
- Use a secrets manager; never commit `.env`.
- Put the API behind HTTPS and a reverse proxy/load balancer.
- Replace the in-process reminder loop with a durable queue/worker (Celery/RQ + Redis/SQS) for multi-instance deployment.
- Add rate limiting, account lockout, audit-log retention policy, malware scanning for uploads, file size limits at the proxy, and monitoring/alerting.
- Add data retention/deletion workflows, candidate consent, privacy notice, access logging, and regional legal review.
- Review matching quality for disparate impact; use skills evidence only and retain human review.
- Add migrations with Alembic before schema changes; this starter uses `create_all` for easy local startup.
- Configure backups/restore drills and automated tests in CI/CD.
