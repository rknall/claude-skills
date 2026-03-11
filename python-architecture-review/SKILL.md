---
name: "python-architecture-review"
description: "Comprehensive design architecture review for Python backend applications. Use this skill when users ask you to review, analyze, or provide feedback on backend architecture designs, system design documents, or Python application architecture. Covers scalability, security, performance, database design, API design, microservices patterns, deployment architecture, and best practices."
---

# Python Backend Architecture Review

Conduct architecture reviews for Python backend applications across infrastructure, code organization, security, scalability, and observability.

## Review Workflow

### Phase 1: Gather Context

Ask the user for:
- Expected scale (users, requests/sec, data volume)
- Performance requirements (latency targets, throughput)
- Security/compliance requirements (GDPR, HIPAA, SOC 2)
- Team size and Python expertise level
- Budget and timeline constraints

If architecture diagrams or documents are provided, identify component boundaries, data flow, external dependencies, and deployment topology.

**Checkpoint:** Do not proceed until you have scale targets and at least one architecture artifact (diagram, doc, or codebase) to review.

### Phase 2: Evaluate Architecture Dimensions

Work through each dimension below. For the full checklist, see [Architecture Checklist](architecture-checklist.md).

#### A. System Architecture & Design Patterns

Evaluate architectural style fit, service boundaries, communication patterns (sync/async, REST/GraphQL/gRPC), and coupling.

Python-specific focus areas:
- Framework choice justification (FastAPI vs Django vs Flask) -- see [Technology Recommendations](technology-recommendations.md)
- ASGI vs WSGI selection
- GIL impact on concurrency strategy
- Async/await usage correctness

Flag: over-engineering for current scale, single points of failure, missing service boundaries.

#### B. Database Architecture

Evaluate schema design, indexing strategy, connection pooling, caching layers, and migration approach.

Example -- SQLAlchemy async connection pooling:

```python
from sqlalchemy.ext.asyncio import create_async_engine, async_sessionmaker

engine = create_async_engine(
    "postgresql+asyncpg://user:pass@localhost/db",
    pool_size=20,
    max_overflow=10,
    pool_timeout=30,
    pool_recycle=1800,
)
AsyncSessionLocal = async_sessionmaker(engine, expire_on_commit=False)

async def get_db():
    async with AsyncSessionLocal() as session:
        yield session
```

Flag: missing connection pooling config, N+1 queries, no caching layer, no migration strategy.

#### C. API Design

Evaluate endpoint consistency, versioning, input validation, error handling, rate limiting, and documentation.

Example -- Pydantic validation with structured errors:

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, EmailStr, field_validator

app = FastAPI()

class UserCreate(BaseModel):
    email: EmailStr
    name: str
    age: int

    @field_validator("age")
    @classmethod
    def age_must_be_positive(cls, v: int) -> int:
        if v < 0 or v > 150:
            raise ValueError("age must be between 0 and 150")
        return v

@app.post("/users", status_code=201)
async def create_user(user: UserCreate):
    # Pydantic validates automatically; 422 on bad input
    return {"id": 1, **user.model_dump()}
```

Flag: inconsistent error formats, missing pagination, no API versioning, no OpenAPI docs.

#### D. Security

Evaluate auth mechanisms, secrets management, input sanitization, encryption (transit + rest), CORS/CSRF config, dependency scanning.

Example -- FastAPI OAuth2 + JWT dependency:

```python
from fastapi import Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer
from jose import JWTError, jwt

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

async def get_current_user(token: str = Depends(oauth2_scheme)):
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        user_id: str = payload.get("sub")
        if user_id is None:
            raise HTTPException(status_code=status.HTTP_401_UNAUTHORIZED)
    except JWTError:
        raise HTTPException(status_code=status.HTTP_401_UNAUTHORIZED)
    return await get_user(user_id)
```

Flag: hardcoded secrets, missing rate limiting, permissive CORS, no audit logging, no dependency scanning.

#### E. Scalability & Performance

Evaluate horizontal/vertical scaling strategy, caching layers, async task processing, load balancing, and resource optimization.

Example -- Celery task with retry:

```python
from celery import Celery

celery_app = Celery("tasks", broker="redis://localhost:6379/0")

@celery_app.task(bind=True, max_retries=3, default_retry_delay=60)
def process_report(self, report_id: int):
    try:
        generate_report(report_id)
    except ExternalServiceError as exc:
        raise self.retry(exc=exc, countdown=2 ** self.request.retries)
```

Flag: synchronous operations that should be async, missing queue infrastructure, no auto-scaling, GIL bottlenecks for CPU-bound work.

#### F. Observability

Evaluate structured logging, metrics collection, distributed tracing, health checks, and alerting.

Example -- health check endpoint:

```python
from fastapi import FastAPI
from sqlalchemy import text

app = FastAPI()

@app.get("/health")
async def health_check(db=Depends(get_db)):
    try:
        await db.execute(text("SELECT 1"))
        return {"status": "healthy", "db": "connected"}
    except Exception:
        return JSONResponse(
            status_code=503,
            content={"status": "unhealthy", "db": "disconnected"},
        )
```

Flag: no structured logging, missing health checks, no distributed tracing, no alerting rules.

#### G. Deployment & Infrastructure

Evaluate containerization, CI/CD pipeline, environment parity, IaC, rollback strategy, and dependency management.

Example -- optimized multi-stage Dockerfile:

```dockerfile
FROM python:3.12-slim AS builder
WORKDIR /app
COPY pyproject.toml poetry.lock ./
RUN pip install poetry && poetry export -f requirements.txt -o requirements.txt

FROM python:3.12-slim
WORKDIR /app
COPY --from=builder /app/requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

Flag: no rollback procedure, manual deployment steps, missing environment parity, unpinned dependencies.

#### H. Code Organization

Evaluate project structure, module boundaries, dependency injection, type hints, and test organization.

Flag: circular imports, missing type hints, no DI pattern, hard-coded configuration values.

#### I. Resilience & Error Handling

Evaluate retry logic, circuit breakers, timeouts, graceful degradation, and dead letter queues.

For implementation patterns, see [Common Patterns](common-patterns.md) (circuit breaker, retry with backoff, event bus).

Flag: no timeouts on external calls, missing circuit breakers, no graceful degradation, inconsistent error handling.

**Checkpoint:** Confirm you have reviewed all dimensions relevant to the user's architecture before writing the report.

### Phase 3: Produce Review Report

Structure your output as follows:

#### 1. Executive Summary
- 1-3 paragraph overall assessment
- Key strengths
- Critical concerns requiring immediate action

#### 2. Detailed Findings

For each reviewed dimension:

> **[Dimension Name]**
>
> **Strengths:** bullet list
>
> **Concerns** (tagged HIGH / MEDIUM / LOW):
> - HIGH: must-fix before production
> - MEDIUM: should-fix soon
> - LOW: nice-to-have
>
> **Recommendations:** specific, actionable items with code examples where helpful

#### 3. Prioritized Next Steps

1. **Must-fix** -- blocking issues with estimated effort
2. **Should-fix** -- important for production readiness
3. **Nice-to-have** -- improvements for later

**Checkpoint:** Before delivering the report, verify every HIGH concern has a concrete recommendation with either a code example or a reference to [Common Patterns](common-patterns.md) or [Technology Recommendations](technology-recommendations.md).

### Phase 4: Interactive Follow-up

After delivering the review:
- Offer to deep-dive into any specific dimension
- Help implement specific recommendations
- Review updated designs after changes are applied

## Reference Materials

- [Architecture Checklist](architecture-checklist.md) -- quick-reference checklist for all review dimensions
- [Common Patterns](common-patterns.md) -- repository pattern, service layer, DI, circuit breaker, CQRS, retry, structured logging
- [Technology Recommendations](technology-recommendations.md) -- framework comparisons, database selection, ORM choices, recommended stack combinations
