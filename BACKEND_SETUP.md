# Polyglot Backend Setup Guide

**Stack:** Python 3.11+ / FastAPI
**Version:** 1.0
**Date:** November 17, 2025

---

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Project Structure](#project-structure)
3. [Initial Setup](#initial-setup)
4. [Configuration](#configuration)
5. [Database Setup](#database-setup)
6. [Running the Application](#running-the-application)
7. [Development Workflow](#development-workflow)
8. [Testing](#testing)
9. [API Documentation](#api-documentation)
10. [Common Commands](#common-commands)

---

## Prerequisites

### Required Software
- **Python 3.11+** - [Download](https://www.python.org/downloads/)
- **PostgreSQL 15+** - [Download](https://www.postgresql.org/download/)
- **Redis 7+** - [Download](https://redis.io/download/)
- **Git** - [Download](https://git-scm.com/downloads)

### Optional but Recommended
- **Docker & Docker Compose** - For containerized development
- **pyenv** - Python version management
- **poetry** - Alternative to pip (dependency management)

### Verify Installation
```bash
python --version  # Should be 3.11 or higher
psql --version    # Should be 15 or higher
redis-server --version  # Should be 7 or higher
```

---

## Project Structure

```
backend/
├── app/
│   ├── __init__.py
│   ├── main.py                 # FastAPI app entry point
│   ├── config.py               # Configuration settings
│   ├── database.py             # Database connection
│   ├── dependencies.py         # Dependency injection
│   │
│   ├── api/                    # API routes
│   │   ├── __init__.py
│   │   ├── v1/
│   │   │   ├── __init__.py
│   │   │   ├── router.py       # API v1 router
│   │   │   ├── auth.py         # Auth endpoints
│   │   │   ├── users.py        # User endpoints
│   │   │   ├── questions.py    # Question endpoints
│   │   │   ├── teams.py        # Team endpoints
│   │   │   └── leaderboard.py  # Leaderboard endpoints
│   │
│   ├── models/                 # SQLAlchemy models
│   │   ├── __init__.py
│   │   ├── user.py
│   │   ├── team.py
│   │   ├── question.py
│   │   ├── answer.py
│   │   ├── response.py
│   │   ├── pilot_group.py
│   │   └── point_transaction.py
│   │
│   ├── schemas/                # Pydantic schemas
│   │   ├── __init__.py
│   │   ├── user.py
│   │   ├── team.py
│   │   ├── question.py
│   │   ├── auth.py
│   │   └── common.py
│   │
│   ├── crud/                   # CRUD operations
│   │   ├── __init__.py
│   │   ├── user.py
│   │   ├── question.py
│   │   ├── team.py
│   │   └── response.py
│   │
│   ├── services/               # Business logic
│   │   ├── __init__.py
│   │   ├── auth_service.py
│   │   ├── question_service.py
│   │   ├── scoring_service.py
│   │   └── pilot_group_service.py
│   │
│   ├── core/                   # Core utilities
│   │   ├── __init__.py
│   │   ├── security.py         # JWT, password hashing
│   │   ├── exceptions.py       # Custom exceptions
│   │   └── middleware.py       # Custom middleware
│   │
│   └── utils/                  # Helper functions
│       ├── __init__.py
│       └── validators.py
│
├── alembic/                    # Database migrations
│   ├── versions/
│   ├── env.py
│   └── script.py.mako
│
├── tests/                      # Test suite
│   ├── __init__.py
│   ├── conftest.py            # Pytest fixtures
│   ├── test_auth.py
│   ├── test_users.py
│   ├── test_questions.py
│   └── test_teams.py
│
├── scripts/                    # Utility scripts
│   ├── seed_data.py           # Seed initial data
│   └── create_admin.py        # Create admin user
│
├── .env.example               # Example environment variables
├── .env                       # Local environment variables (gitignored)
├── .gitignore
├── requirements.txt           # Production dependencies
├── requirements-dev.txt       # Development dependencies
├── pytest.ini                 # Pytest configuration
├── alembic.ini               # Alembic configuration
├── docker-compose.yml        # Docker services
├── Dockerfile                # Docker image
└── README.md                 # Backend documentation
```

---

## Initial Setup

### Step 1: Clone Repository (if not already done)
```bash
git clone <repository-url>
cd Polyglot
```

### Step 2: Create Backend Directory
```bash
mkdir -p backend
cd backend
```

### Step 3: Create Virtual Environment
```bash
# Using venv
python -m venv venv

# Activate virtual environment
# On macOS/Linux:
source venv/bin/activate
# On Windows:
venv\Scripts\activate
```

### Step 4: Create Requirements Files

**requirements.txt** (Production dependencies):
```txt
# FastAPI and ASGI server
fastapi==0.104.1
uvicorn[standard]==0.24.0
python-multipart==0.0.6

# Database
sqlalchemy[asyncio]==2.0.23
asyncpg==0.29.0  # Async PostgreSQL driver
alembic==1.12.1

# Redis
redis==5.0.1
celery==5.3.4

# Authentication
python-jose[cryptography]==3.3.0
passlib[bcrypt]==1.7.4
python-dotenv==1.0.0

# Validation
pydantic==2.5.0
pydantic-settings==2.1.0
email-validator==2.1.0

# CORS
python-cors==1.0.0

# Utilities
python-dateutil==2.8.2
```

**requirements-dev.txt** (Development dependencies):
```txt
-r requirements.txt

# Testing
pytest==7.4.3
pytest-asyncio==0.21.1
pytest-cov==4.1.0
httpx==0.25.2  # For TestClient

# Code quality
black==23.12.0
ruff==0.1.7
mypy==1.7.1

# Development tools
ipython==8.18.1
```

### Step 5: Install Dependencies
```bash
pip install --upgrade pip
pip install -r requirements-dev.txt
```

---

## Configuration

### Create .env File
```bash
cp .env.example .env
```

### .env.example
```bash
# Application
APP_NAME=Polyglot
ENVIRONMENT=development
DEBUG=True
API_V1_PREFIX=/api/v1

# Server
HOST=0.0.0.0
PORT=8000

# Database
DATABASE_URL=postgresql+asyncpg://polyglot:dev_password@localhost:5432/polyglot_dev
DATABASE_POOL_SIZE=20
DATABASE_MAX_OVERFLOW=0

# Redis
REDIS_URL=redis://localhost:6379/0

# JWT Authentication
SECRET_KEY=your-secret-key-change-this-in-production
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=15
REFRESH_TOKEN_EXPIRE_DAYS=7

# CORS
CORS_ORIGINS=["http://localhost:5173","http://localhost:3000"]

# Rate Limiting
RATE_LIMIT_ENABLED=True
RATE_LIMIT_REQUESTS=100
RATE_LIMIT_WINDOW=900  # 15 minutes in seconds

# Celery
CELERY_BROKER_URL=redis://localhost:6379/1
CELERY_RESULT_BACKEND=redis://localhost:6379/2

# Pilot Groups
PILOT_GROUP_SIZE=100
PILOT_GROUP_MIN_SIZE=10

# Scoring
BASE_POINTS=10
```

### app/config.py
```python
from pydantic_settings import BaseSettings, SettingsConfigDict
from typing import List


class Settings(BaseSettings):
    # Application
    APP_NAME: str = "Polyglot"
    ENVIRONMENT: str = "development"
    DEBUG: bool = True
    API_V1_PREFIX: str = "/api/v1"

    # Server
    HOST: str = "0.0.0.0"
    PORT: int = 8000

    # Database
    DATABASE_URL: str
    DATABASE_POOL_SIZE: int = 20
    DATABASE_MAX_OVERFLOW: int = 0

    # Redis
    REDIS_URL: str

    # JWT
    SECRET_KEY: str
    ALGORITHM: str = "HS256"
    ACCESS_TOKEN_EXPIRE_MINUTES: int = 15
    REFRESH_TOKEN_EXPIRE_DAYS: int = 7

    # CORS
    CORS_ORIGINS: List[str] = ["http://localhost:5173"]

    # Rate Limiting
    RATE_LIMIT_ENABLED: bool = True
    RATE_LIMIT_REQUESTS: int = 100
    RATE_LIMIT_WINDOW: int = 900

    # Celery
    CELERY_BROKER_URL: str
    CELERY_RESULT_BACKEND: str

    # Pilot Groups
    PILOT_GROUP_SIZE: int = 100
    PILOT_GROUP_MIN_SIZE: int = 10

    # Scoring
    BASE_POINTS: int = 10

    model_config = SettingsConfigDict(
        env_file=".env",
        case_sensitive=True
    )


settings = Settings()
```

---

## Database Setup

### Step 1: Create Database
```bash
# Connect to PostgreSQL
psql -U postgres

# Create database and user
CREATE DATABASE polyglot_dev;
CREATE USER polyglot WITH PASSWORD 'dev_password';
GRANT ALL PRIVILEGES ON DATABASE polyglot_dev TO polyglot;
\q
```

### Step 2: Setup Alembic
```bash
# Initialize Alembic (if not already done)
alembic init alembic
```

### alembic.ini (Update sqlalchemy.url)
```ini
# Comment out the default sqlalchemy.url
# sqlalchemy.url = driver://user:pass@localhost/dbname

# We'll set it programmatically from .env
```

### alembic/env.py (Update for async)
```python
from logging.config import fileConfig
from sqlalchemy import pool
from sqlalchemy.engine import Connection
from sqlalchemy.ext.asyncio import async_engine_from_config
from alembic import context

# Import your models and settings
from app.config import settings
from app.database import Base
from app.models import *  # Import all models

config = context.config

# Set sqlalchemy.url from settings
config.set_main_option("sqlalchemy.url", settings.DATABASE_URL)

if config.config_file_name is not None:
    fileConfig(config.config_file_name)

target_metadata = Base.metadata


def run_migrations_offline() -> None:
    url = config.get_main_option("sqlalchemy.url")
    context.configure(
        url=url,
        target_metadata=target_metadata,
        literal_binds=True,
        dialect_opts={"paramstyle": "named"},
    )

    with context.begin_transaction():
        context.run_migrations()


def do_run_migrations(connection: Connection) -> None:
    context.configure(connection=connection, target_metadata=target_metadata)

    with context.begin_transaction():
        context.run_migrations()


async def run_async_migrations() -> None:
    connectable = async_engine_from_config(
        config.get_section(config.config_ini_section, {}),
        prefix="sqlalchemy.",
        poolclass=pool.NullPool,
    )

    async with connectable.connect() as connection:
        await connection.run_sync(do_run_migrations)

    await connectable.dispose()


def run_migrations_online() -> None:
    import asyncio
    asyncio.run(run_async_migrations())


if context.is_offline_mode():
    run_migrations_offline()
else:
    run_migrations_online()
```

### Step 3: Create Database Connection (app/database.py)
```python
from sqlalchemy.ext.asyncio import AsyncSession, create_async_engine
from sqlalchemy.orm import sessionmaker, declarative_base
from app.config import settings

# Create async engine
engine = create_async_engine(
    settings.DATABASE_URL,
    echo=settings.DEBUG,
    pool_size=settings.DATABASE_POOL_SIZE,
    max_overflow=settings.DATABASE_MAX_OVERFLOW,
)

# Create async session factory
AsyncSessionLocal = sessionmaker(
    engine,
    class_=AsyncSession,
    expire_on_commit=False,
)

# Create declarative base
Base = declarative_base()


# Dependency for getting DB session
async def get_db() -> AsyncSession:
    async with AsyncSessionLocal() as session:
        try:
            yield session
        finally:
            await session.close()
```

### Step 4: Create Initial Migration
```bash
# Create first migration
alembic revision --autogenerate -m "Initial migration"

# Apply migration
alembic upgrade head
```

---

## Running the Application

### Using Docker Compose (Recommended for Development)

**docker-compose.yml**:
```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: polyglot_dev
      POSTGRES_USER: polyglot
      POSTGRES_PASSWORD: dev_password
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U polyglot"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 5

  backend:
    build: .
    command: uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
    volumes:
      - .:/app
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql+asyncpg://polyglot:dev_password@postgres:5432/polyglot_dev
      - REDIS_URL=redis://redis:6379/0
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy

  celery_worker:
    build: .
    command: celery -A app.tasks worker --loglevel=info
    volumes:
      - .:/app
    environment:
      - DATABASE_URL=postgresql+asyncpg://polyglot:dev_password@postgres:5432/polyglot_dev
      - CELERY_BROKER_URL=redis://redis:6379/1
      - CELERY_RESULT_BACKEND=redis://redis:6379/2
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy

volumes:
  postgres_data:
  redis_data:
```

**Dockerfile**:
```dockerfile
FROM python:3.11-slim

WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    gcc \
    postgresql-client \
    && rm -rf /var/lib/apt/lists/*

# Install Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Expose port
EXPOSE 8000

# Run application
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

**Start Services**:
```bash
# Start all services
docker-compose up -d

# View logs
docker-compose logs -f backend

# Stop services
docker-compose down
```

### Without Docker

**Terminal 1 - PostgreSQL** (if not running as service):
```bash
# Start PostgreSQL (if installed locally)
# macOS with Homebrew:
brew services start postgresql

# Linux:
sudo systemctl start postgresql
```

**Terminal 2 - Redis**:
```bash
redis-server
```

**Terminal 3 - FastAPI Application**:
```bash
cd backend
source venv/bin/activate
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

**Terminal 4 - Celery Worker** (for background tasks):
```bash
cd backend
source venv/bin/activate
celery -A app.tasks worker --loglevel=info
```

### Access the Application
- **API:** http://localhost:8000
- **API Docs (Swagger UI):** http://localhost:8000/docs
- **API Docs (ReDoc):** http://localhost:8000/redoc
- **OpenAPI JSON:** http://localhost:8000/openapi.json

---

## Development Workflow

### Daily Workflow
```bash
# 1. Activate virtual environment
source venv/bin/activate

# 2. Pull latest changes
git pull

# 3. Install any new dependencies
pip install -r requirements-dev.txt

# 4. Run migrations
alembic upgrade head

# 5. Start development server
uvicorn app.main:app --reload

# 6. Make your changes...

# 7. Run tests
pytest

# 8. Format code
black app/
ruff check app/ --fix

# 9. Commit changes
git add .
git commit -m "Your commit message"
git push
```

### Creating New Features
```bash
# 1. Create feature branch
git checkout -b feature/your-feature-name

# 2. Make changes...

# 3. Create migration if models changed
alembic revision --autogenerate -m "Add new field to user model"
alembic upgrade head

# 4. Write tests
# tests/test_your_feature.py

# 5. Run tests
pytest tests/test_your_feature.py

# 6. Push and create PR
git push origin feature/your-feature-name
```

---

## Testing

### Run All Tests
```bash
pytest
```

### Run with Coverage
```bash
pytest --cov=app --cov-report=html --cov-report=term
```

### Run Specific Test File
```bash
pytest tests/test_auth.py
```

### Run Specific Test Function
```bash
pytest tests/test_auth.py::test_user_registration
```

### Example Test (tests/conftest.py)
```python
import pytest
from httpx import AsyncClient
from sqlalchemy.ext.asyncio import AsyncSession, create_async_engine
from sqlalchemy.orm import sessionmaker
from app.main import app
from app.database import Base, get_db

# Test database URL
TEST_DATABASE_URL = "postgresql+asyncpg://polyglot:dev_password@localhost:5432/polyglot_test"

# Create test engine
test_engine = create_async_engine(TEST_DATABASE_URL, echo=False)
TestAsyncSessionLocal = sessionmaker(
    test_engine, class_=AsyncSession, expire_on_commit=False
)


@pytest.fixture(scope="function")
async def test_db():
    # Create tables
    async with test_engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)

    # Provide session
    async with TestAsyncSessionLocal() as session:
        yield session

    # Drop tables
    async with test_engine.begin() as conn:
        await conn.run_sync(Base.metadata.drop_all)


@pytest.fixture(scope="function")
async def client(test_db):
    async def override_get_db():
        yield test_db

    app.dependency_overrides[get_db] = override_get_db

    async with AsyncClient(app=app, base_url="http://test") as ac:
        yield ac

    app.dependency_overrides.clear()
```

---

## API Documentation

FastAPI automatically generates interactive API documentation:

### Swagger UI
Visit: http://localhost:8000/docs

Features:
- Interactive API explorer
- Try out endpoints directly
- View request/response schemas
- Authentication support

### ReDoc
Visit: http://localhost:8000/redoc

Features:
- Clean, readable documentation
- Better for sharing with frontend team
- Printable format

---

## Common Commands

### Database
```bash
# Create migration
alembic revision --autogenerate -m "Description"

# Apply migrations
alembic upgrade head

# Rollback one migration
alembic downgrade -1

# View migration history
alembic history

# Seed database
python scripts/seed_data.py
```

### Testing
```bash
# Run all tests
pytest

# Run with coverage
pytest --cov

# Run specific tests
pytest tests/test_auth.py

# Run tests matching pattern
pytest -k "user"
```

### Code Quality
```bash
# Format code with Black
black app/

# Lint with Ruff
ruff check app/

# Fix linting issues
ruff check app/ --fix

# Type checking with mypy
mypy app/
```

### Docker
```bash
# Build and start
docker-compose up --build

# Start in background
docker-compose up -d

# View logs
docker-compose logs -f backend

# Stop services
docker-compose down

# Remove volumes
docker-compose down -v
```

---

## Next Steps

1. ✅ **Setup Complete** - You should now have a running backend
2. **Implement Models** - Create SQLAlchemy models for all entities
3. **Create Schemas** - Define Pydantic schemas for validation
4. **Build API Endpoints** - Implement REST endpoints
5. **Add Authentication** - Implement JWT auth system
6. **Write Tests** - Add comprehensive test coverage
7. **Integrate Frontend** - Connect React frontend

---

## Troubleshooting

### Issue: Database connection error
```bash
# Check PostgreSQL is running
psql -U postgres -c "SELECT version();"

# Check database exists
psql -U postgres -l | grep polyglot
```

### Issue: Module not found
```bash
# Reinstall dependencies
pip install -r requirements-dev.txt

# Check virtual environment is activated
which python  # Should point to venv
```

### Issue: Migration conflicts
```bash
# Drop all tables and recreate
alembic downgrade base
alembic upgrade head
```

### Issue: Port already in use
```bash
# Find process using port 8000
lsof -i :8000

# Kill process
kill -9 <PID>
```

---

## Related Documentation
- [Technical Specifications](TECHNICAL_SPECS.md)
- [Project Plan](PROJECT_PLAN.md)
- [MVP Requirements](MVP_REQUIREMENTS.md)

---

**Author:** Claude
**Last Updated:** November 17, 2025
**Version:** 1.0
