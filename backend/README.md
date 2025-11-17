# Polyglot Backend

FastAPI backend for the Polyglot gamified learning platform.

## Quick Start

### Prerequisites
- Python 3.11+
- PostgreSQL 15+
- Redis 7+

### Setup

1. Create virtual environment:
```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

2. Install dependencies:
```bash
pip install -r requirements-dev.txt
```

3. Setup environment:
```bash
cp .env.example .env
# Edit .env with your configuration
```

4. Run database migrations:
```bash
alembic upgrade head
```

5. Start development server:
```bash
uvicorn app.main:app --reload
```

## Documentation

- **API Docs (Swagger):** http://localhost:8000/docs
- **API Docs (ReDoc):** http://localhost:8000/redoc
- **Setup Guide:** [BACKEND_SETUP.md](../BACKEND_SETUP.md)
- **Technical Specs:** [TECHNICAL_SPECS.md](../TECHNICAL_SPECS.md)

## Project Structure

```
backend/
├── app/           # Application code
├── tests/         # Test suite
├── scripts/       # Utility scripts
└── alembic/       # Database migrations
```

## Testing

```bash
# Run all tests
pytest

# Run with coverage
pytest --cov

# Run specific test
pytest tests/test_auth.py
```

## Development

```bash
# Format code
black app/

# Lint code
ruff check app/

# Type checking
mypy app/
```

## License

TBD
