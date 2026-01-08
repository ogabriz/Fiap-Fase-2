# Copilot Instructions for ToggleMaster Monorepo

This repository contains multiple services for feature flag management, targeting, evaluation, analytics, and authentication. Each service is isolated in its own folder and communicates via HTTP APIs and AWS resources.

## Architecture Overview
- **Services:**
  - `auth-service` (Go): API key management and authentication. PostgreSQL backend.
  - `flag-service` (Python): CRUD for feature flags. Protected by API keys from `auth-service`. PostgreSQL backend.
  - `targeting-service` (Python): Manages targeting rules for flags. Protected by API keys from `auth-service`. PostgreSQL backend.
  - `evaluation-service` (Go): Hot path for flag evaluation. Uses Redis for caching, fetches flag/rule data from other services, and sends events to AWS SQS.
  - `analytics-service` (Python): Worker that consumes SQS events and writes analytics to DynamoDB.

## Service Boundaries & Data Flow
- **API keys** are created in `auth-service` and used as Bearer tokens for all protected endpoints in other services.
- **flag-service** and **targeting-service** expose CRUD APIs, require valid API keys, and depend on `auth-service` for authentication.
- **evaluation-service** is the only public endpoint for clients. It fetches flag/rule data from other services, caches results in Redis, and emits events to SQS.
- **analytics-service** consumes SQS events and writes to DynamoDB. No public API except `/health`.

## Developer Workflows
- **Environment Setup:**
  - Each service requires its own `.env` file (see each service's README for required variables).
  - PostgreSQL must be running for `auth-service`, `flag-service`, and `targeting-service`.
  - Redis required for `evaluation-service`.
  - AWS credentials required for `evaluation-service` (SQS) and `analytics-service` (SQS/DynamoDB).
- **Build & Run:**
  - Go services: `go mod tidy` then `go run .`
  - Python services: `pip install -r requirements.txt` then `gunicorn --bind 0.0.0.0:<PORT> app:app`
- **Database Initialization:**
  - Run the appropriate `db/init.sql` for each service's database.
- **Testing Endpoints:**
  - Use `curl` commands from READMEs to test health and CRUD endpoints. Always include `Authorization: Bearer <API_KEY>` for protected endpoints.

## Conventions & Patterns
- **API Authentication:** All non-health endpoints require Bearer tokens from `auth-service`.
- **Service URLs:** Hardcoded in `.env` files; update as needed for local or cloud deployments.
- **Event Flow:**
  - `evaluation-service` emits events to SQS after each evaluation.
  - `analytics-service` listens to SQS and writes to DynamoDB.
- **Caching:** `evaluation-service` uses Redis for fast flag/rule lookup.
- **Health Checks:** All services expose `/health` endpoints returning `{"status":"ok"}`.

## Integration Points
- **PostgreSQL:** Used by `auth-service`, `flag-service`, `targeting-service`.
- **Redis:** Used by `evaluation-service` for caching.
- **AWS SQS/DynamoDB:** Used for event flow and analytics.

## Key Files & Examples
- See each service's `README.md` for setup, environment variables, and example API calls.
- Database schemas: `db/init.sql` in each service.
- Main entrypoints: `app.py` (Python), `main.go` (Go).

## Quickstart Example
1. Start `auth-service` and create an API key.
2. Start `flag-service` and `targeting-service` with the API key.
3. Start `evaluation-service` (with Redis and AWS SQS configured).
4. Start `analytics-service` (with AWS credentials and DynamoDB configured).
5. Use `curl` commands from READMEs to interact with services.

---
For more details, always refer to the individual service's `README.md`.
