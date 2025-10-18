# Common Python Backend Architecture Patterns

This document provides reference implementations and patterns for common architectural decisions.

## 1. Repository Pattern

```python
from abc import ABC, abstractmethod
from typing import Generic, TypeVar, Optional, List
from sqlalchemy.orm import Session

T = TypeVar('T')

class BaseRepository(ABC, Generic[T]):
    """Abstract base repository for data access"""

    @abstractmethod
    async def get(self, id: int) -> Optional[T]:
        pass

    @abstractmethod
    async def list(self, skip: int = 0, limit: int = 100) -> List[T]:
        pass

    @abstractmethod
    async def create(self, obj: T) -> T:
        pass

    @abstractmethod
    async def update(self, id: int, obj: T) -> Optional[T]:
        pass

    @abstractmethod
    async def delete(self, id: int) -> bool:
        pass


class SQLAlchemyRepository(BaseRepository[T]):
    """SQLAlchemy implementation of repository pattern"""

    def __init__(self, session: Session, model_class: type):
        self.session = session
        self.model_class = model_class

    async def get(self, id: int) -> Optional[T]:
        return self.session.query(self.model_class).filter_by(id=id).first()

    async def list(self, skip: int = 0, limit: int = 100) -> List[T]:
        return self.session.query(self.model_class).offset(skip).limit(limit).all()

    async def create(self, obj: T) -> T:
        self.session.add(obj)
        self.session.commit()
        self.session.refresh(obj)
        return obj

    async def update(self, id: int, obj: T) -> Optional[T]:
        existing = await self.get(id)
        if existing:
            for key, value in obj.__dict__.items():
                setattr(existing, key, value)
            self.session.commit()
            return existing
        return None

    async def delete(self, id: int) -> bool:
        obj = await self.get(id)
        if obj:
            self.session.delete(obj)
            self.session.commit()
            return True
        return False
```

## 2. Service Layer Pattern

```python
from typing import Protocol
from .repositories import UserRepository
from .models import User
from .schemas import UserCreate, UserUpdate

class IUserService(Protocol):
    """Interface for user service"""

    async def get_user(self, user_id: int) -> User:
        ...

    async def create_user(self, user_data: UserCreate) -> User:
        ...


class UserService:
    """Service layer for user business logic"""

    def __init__(self, user_repo: UserRepository):
        self.user_repo = user_repo

    async def get_user(self, user_id: int) -> User:
        user = await self.user_repo.get(user_id)
        if not user:
            raise ValueError(f"User {user_id} not found")
        return user

    async def create_user(self, user_data: UserCreate) -> User:
        # Business logic here
        if await self._email_exists(user_data.email):
            raise ValueError("Email already registered")

        user = User(**user_data.dict())
        return await self.user_repo.create(user)

    async def _email_exists(self, email: str) -> bool:
        # Check if email exists
        return await self.user_repo.find_by_email(email) is not None
```

## 3. Dependency Injection with FastAPI

```python
from fastapi import Depends, FastAPI
from sqlalchemy.orm import Session
from typing import Generator

app = FastAPI()

# Database session dependency
def get_db() -> Generator[Session, None, None]:
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

# Repository dependency
def get_user_repository(db: Session = Depends(get_db)) -> UserRepository:
    return UserRepository(db)

# Service dependency
def get_user_service(
    user_repo: UserRepository = Depends(get_user_repository)
) -> UserService:
    return UserService(user_repo)

# Route using dependency injection
@app.get("/users/{user_id}")
async def get_user(
    user_id: int,
    user_service: UserService = Depends(get_user_service)
):
    return await user_service.get_user(user_id)
```

## 4. Event-Driven Architecture

```python
from typing import Callable, Dict, List, Any
from dataclasses import dataclass
from datetime import datetime
import asyncio

@dataclass
class Event:
    """Base event class"""
    event_type: str
    data: Dict[str, Any]
    timestamp: datetime = datetime.utcnow()

class EventBus:
    """Simple in-memory event bus"""

    def __init__(self):
        self._subscribers: Dict[str, List[Callable]] = {}

    def subscribe(self, event_type: str, handler: Callable):
        """Subscribe to an event type"""
        if event_type not in self._subscribers:
            self._subscribers[event_type] = []
        self._subscribers[event_type].append(handler)

    async def publish(self, event: Event):
        """Publish an event to all subscribers"""
        handlers = self._subscribers.get(event.event_type, [])
        await asyncio.gather(*[handler(event) for handler in handlers])

# Usage
event_bus = EventBus()

@dataclass
class UserCreatedEvent(Event):
    event_type: str = "user.created"

async def send_welcome_email(event: Event):
    print(f"Sending welcome email for user {event.data['user_id']}")

async def create_user_profile(event: Event):
    print(f"Creating profile for user {event.data['user_id']}")

# Subscribe handlers
event_bus.subscribe("user.created", send_welcome_email)
event_bus.subscribe("user.created", create_user_profile)

# Publish event
await event_bus.publish(UserCreatedEvent(data={"user_id": 123}))
```

## 5. Circuit Breaker Pattern

```python
from enum import Enum
from datetime import datetime, timedelta
from typing import Callable, Any
import asyncio

class CircuitState(Enum):
    CLOSED = "closed"
    OPEN = "open"
    HALF_OPEN = "half_open"

class CircuitBreaker:
    """Circuit breaker for external service calls"""

    def __init__(
        self,
        failure_threshold: int = 5,
        timeout: int = 60,
        recovery_timeout: int = 30
    ):
        self.failure_threshold = failure_threshold
        self.timeout = timeout
        self.recovery_timeout = recovery_timeout
        self.failure_count = 0
        self.last_failure_time = None
        self.state = CircuitState.CLOSED

    async def call(self, func: Callable, *args, **kwargs) -> Any:
        """Execute function with circuit breaker protection"""

        if self.state == CircuitState.OPEN:
            if self._should_attempt_reset():
                self.state = CircuitState.HALF_OPEN
            else:
                raise Exception("Circuit breaker is OPEN")

        try:
            result = await asyncio.wait_for(
                func(*args, **kwargs),
                timeout=self.timeout
            )
            self._on_success()
            return result
        except Exception as e:
            self._on_failure()
            raise e

    def _on_success(self):
        """Handle successful call"""
        self.failure_count = 0
        self.state = CircuitState.CLOSED

    def _on_failure(self):
        """Handle failed call"""
        self.failure_count += 1
        self.last_failure_time = datetime.utcnow()

        if self.failure_count >= self.failure_threshold:
            self.state = CircuitState.OPEN

    def _should_attempt_reset(self) -> bool:
        """Check if enough time has passed to retry"""
        if self.last_failure_time is None:
            return True

        return (
            datetime.utcnow() - self.last_failure_time
        ).total_seconds() >= self.recovery_timeout

# Usage
circuit_breaker = CircuitBreaker(failure_threshold=3, timeout=5)

async def call_external_api():
    # External API call
    pass

result = await circuit_breaker.call(call_external_api)
```

## 6. CQRS Pattern

```python
from abc import ABC, abstractmethod
from typing import Generic, TypeVar
from pydantic import BaseModel

# Commands
class Command(BaseModel):
    """Base command"""
    pass

class CreateUserCommand(Command):
    email: str
    name: str

# Command Handlers
TCommand = TypeVar('TCommand', bound=Command)

class CommandHandler(ABC, Generic[TCommand]):
    """Abstract command handler"""

    @abstractmethod
    async def handle(self, command: TCommand) -> Any:
        pass

class CreateUserCommandHandler(CommandHandler[CreateUserCommand]):
    """Handler for creating users"""

    def __init__(self, user_repo: UserRepository):
        self.user_repo = user_repo

    async def handle(self, command: CreateUserCommand) -> User:
        user = User(email=command.email, name=command.name)
        return await self.user_repo.create(user)

# Queries
class Query(BaseModel):
    """Base query"""
    pass

class GetUserQuery(Query):
    user_id: int

# Query Handlers
TQuery = TypeVar('TQuery', bound=Query)

class QueryHandler(ABC, Generic[TQuery]):
    """Abstract query handler"""

    @abstractmethod
    async def handle(self, query: TQuery) -> Any:
        pass

class GetUserQueryHandler(QueryHandler[GetUserQuery]):
    """Handler for getting users"""

    def __init__(self, user_repo: UserRepository):
        self.user_repo = user_repo

    async def handle(self, query: GetUserQuery) -> User:
        return await self.user_repo.get(query.user_id)

# Command Bus
class CommandBus:
    """Simple command bus"""

    def __init__(self):
        self._handlers = {}

    def register(self, command_type: type, handler: CommandHandler):
        self._handlers[command_type] = handler

    async def execute(self, command: Command):
        handler = self._handlers.get(type(command))
        if not handler:
            raise ValueError(f"No handler for {type(command)}")
        return await handler.handle(command)
```

## 7. Retry Pattern with Exponential Backoff

```python
import asyncio
from typing import Callable, TypeVar, Any
from functools import wraps

T = TypeVar('T')

def retry_with_backoff(
    max_retries: int = 3,
    base_delay: float = 1.0,
    max_delay: float = 60.0,
    exponential_base: float = 2.0
):
    """Decorator for retry logic with exponential backoff"""

    def decorator(func: Callable[..., T]) -> Callable[..., T]:
        @wraps(func)
        async def wrapper(*args, **kwargs) -> T:
            retries = 0

            while retries < max_retries:
                try:
                    return await func(*args, **kwargs)
                except Exception as e:
                    retries += 1

                    if retries >= max_retries:
                        raise e

                    delay = min(
                        base_delay * (exponential_base ** (retries - 1)),
                        max_delay
                    )

                    print(f"Retry {retries}/{max_retries} after {delay}s")
                    await asyncio.sleep(delay)

            raise Exception("Max retries exceeded")

        return wrapper
    return decorator

# Usage
@retry_with_backoff(max_retries=3, base_delay=1.0)
async def fetch_data_from_api():
    # API call that might fail
    pass
```

## 8. Settings Management

```python
from pydantic_settings import BaseSettings
from functools import lru_cache

class Settings(BaseSettings):
    """Application settings"""

    # API Settings
    api_title: str = "My API"
    api_version: str = "1.0.0"

    # Database
    database_url: str
    db_pool_size: int = 5

    # Redis
    redis_url: str
    redis_ttl: int = 3600

    # Security
    secret_key: str
    algorithm: str = "HS256"
    access_token_expire_minutes: int = 30

    # External Services
    external_api_url: str
    external_api_key: str

    # Observability
    log_level: str = "INFO"
    sentry_dsn: str | None = None

    class Config:
        env_file = ".env"
        case_sensitive = False

@lru_cache()
def get_settings() -> Settings:
    """Cached settings instance"""
    return Settings()

# Usage
settings = get_settings()
```

## 9. Background Task Processing

```python
from celery import Celery
from typing import Any

# Celery app
celery_app = Celery(
    "tasks",
    broker="redis://localhost:6379/0",
    backend="redis://localhost:6379/0"
)

celery_app.conf.update(
    task_serializer="json",
    accept_content=["json"],
    result_serializer="json",
    timezone="UTC",
    enable_utc=True,
    task_track_started=True,
    task_time_limit=300,  # 5 minutes
    task_soft_time_limit=240,  # 4 minutes
)

@celery_app.task(bind=True, max_retries=3)
def process_data(self, data: dict) -> Any:
    """Background task with retry logic"""
    try:
        # Process data
        result = heavy_computation(data)
        return result
    except Exception as exc:
        # Retry with exponential backoff
        raise self.retry(exc=exc, countdown=2 ** self.request.retries)

# FastAPI integration
from fastapi import FastAPI, BackgroundTasks

app = FastAPI()

@app.post("/process")
async def trigger_processing(data: dict, background_tasks: BackgroundTasks):
    # Option 1: FastAPI background tasks (for quick tasks)
    background_tasks.add_task(quick_task, data)

    # Option 2: Celery (for long-running tasks)
    task = process_data.delay(data)

    return {"task_id": task.id}

@app.get("/task/{task_id}")
async def get_task_status(task_id: str):
    task = celery_app.AsyncResult(task_id)
    return {
        "task_id": task_id,
        "status": task.status,
        "result": task.result if task.ready() else None
    }
```

## 10. API Versioning

```python
from fastapi import APIRouter, FastAPI
from enum import Enum

class APIVersion(str, Enum):
    V1 = "v1"
    V2 = "v2"

app = FastAPI()

# Version 1 router
router_v1 = APIRouter(prefix="/api/v1", tags=["v1"])

@router_v1.get("/users/{user_id}")
async def get_user_v1(user_id: int):
    return {"id": user_id, "version": "v1"}

# Version 2 router
router_v2 = APIRouter(prefix="/api/v2", tags=["v2"])

@router_v2.get("/users/{user_id}")
async def get_user_v2(user_id: int):
    return {
        "id": user_id,
        "version": "v2",
        "additional_field": "new in v2"
    }

app.include_router(router_v1)
app.include_router(router_v2)

# Header-based versioning (alternative)
from fastapi import Header

@app.get("/users/{user_id}")
async def get_user(
    user_id: int,
    api_version: str = Header(default="v1", alias="X-API-Version")
):
    if api_version == "v2":
        return {"id": user_id, "version": "v2"}
    return {"id": user_id, "version": "v1"}
```

## 11. Middleware Patterns

```python
from fastapi import FastAPI, Request
from starlette.middleware.base import BaseHTTPMiddleware
import time
import logging

class LoggingMiddleware(BaseHTTPMiddleware):
    """Middleware for request/response logging"""

    async def dispatch(self, request: Request, call_next):
        start_time = time.time()

        # Log request
        logging.info(f"Request: {request.method} {request.url}")

        response = await call_next(request)

        # Log response
        process_time = time.time() - start_time
        logging.info(
            f"Response: {response.status_code} "
            f"(took {process_time:.2f}s)"
        )

        response.headers["X-Process-Time"] = str(process_time)
        return response

class RateLimitMiddleware(BaseHTTPMiddleware):
    """Simple rate limiting middleware"""

    def __init__(self, app, requests_per_minute: int = 60):
        super().__init__(app)
        self.requests_per_minute = requests_per_minute
        self.request_counts = {}

    async def dispatch(self, request: Request, call_next):
        client_ip = request.client.host
        current_minute = int(time.time() / 60)

        key = f"{client_ip}:{current_minute}"
        self.request_counts[key] = self.request_counts.get(key, 0) + 1

        if self.request_counts[key] > self.requests_per_minute:
            return JSONResponse(
                status_code=429,
                content={"error": "Rate limit exceeded"}
            )

        return await call_next(request)

# Add middleware to app
app = FastAPI()
app.add_middleware(LoggingMiddleware)
app.add_middleware(RateLimitMiddleware, requests_per_minute=100)
```

## 12. Structured Logging

```python
import structlog
from typing import Any
import logging

# Configure structlog
structlog.configure(
    processors=[
        structlog.stdlib.filter_by_level,
        structlog.stdlib.add_logger_name,
        structlog.stdlib.add_log_level,
        structlog.stdlib.PositionalArgumentsFormatter(),
        structlog.processors.TimeStamper(fmt="iso"),
        structlog.processors.StackInfoRenderer(),
        structlog.processors.format_exc_info,
        structlog.processors.UnicodeDecoder(),
        structlog.processors.JSONRenderer()
    ],
    context_class=dict,
    logger_factory=structlog.stdlib.LoggerFactory(),
    cache_logger_on_first_use=True,
)

# Get logger
logger = structlog.get_logger()

# Usage
logger.info(
    "user_created",
    user_id=123,
    email="user@example.com",
    ip_address="192.168.1.1"
)

# Context binding
logger = logger.bind(request_id="abc-123")
logger.info("processing_request")
logger.info("request_completed", duration_ms=150)
```

These patterns provide battle-tested solutions for common architectural challenges in Python backend development.
