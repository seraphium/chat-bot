# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a full-stack AI chatbot application with:
- **Frontend**: Next.js 15 + React 19 + TypeScript + Tailwind CSS
- **Backend**: FastAPI + Python 3.10+ + LangChain + SQLAlchemy 2.0
- **Database**: SQLite (dev) / PostgreSQL (prod)
- **Cache**: Redis
- **Task Queue**: Celery with Flower monitoring
- **Vector Store**: ChromaDB

## Development Commands

### Frontend (bolt-chatbot-ui)
```bash
cd bolt-chatbot-ui

# Development
npm run dev              # Start dev server (localhost:3000)

# Build & Production
npm run build           # Build for production
npm run start           # Start production server

# Code Quality
npm run lint            # Run ESLint
npm run type-check      # TypeScript type checking (manual)
```

### Backend (langchain_backend)
```bash
cd langchain_backend

# Development (Poetry)
poetry install          # Install dependencies
poetry run python -m app.main  # Start dev server (localhost:8000)

# Alternative (pip)
pip install -r requirements.txt
python -m app.main

# Testing
poetry run pytest              # Run all tests
poetry run pytest tests/unit/  # Unit tests only
poetry run pytest tests/integration/  # Integration tests
poetry run pytest tests/e2e/   # End-to-end tests
poetry run pytest --cov=app    # With coverage

# Background Services
poetry run python start_celery_worker.py  # Start Celery worker
poetry run python start_flower.py         # Start Flower monitoring (localhost:5555)

# Code Quality
poetry run black app/           # Format Python code
poetry run flake8 app/          # Lint Python code
```

### Docker Deployment
```bash
# Frontend only
docker-compose up -d

# Full stack (requires backend Docker setup)
docker-compose -f docker-compose.full.yml up -d
```

## Architecture Highlights

### Frontend Structure
- `app/` - Next.js App Router pages
  - `(auth)/` - Authentication pages (login/register)
  - `api/` - API routes
- `components/` - React components
  - `auth/` - Auth components
  - `ui/` - Base UI components (Radix UI)
  - Chat components: `ChatHistory.tsx`, `ChatInput.tsx`, `ChatMessage.tsx`
- `lib/` - Utilities and API client
- `hooks/` - Custom React hooks

### Backend Structure
- `app/` - Main application
  - `api/` - API routes and middleware
    - `routes/` - Route handlers (auth, chat, file, health)
    - `middleware/` - CORS, logging middleware
  - `chains/` - LangChain implementations
    - `managers/` - Cache and chain managers
  - `models/` - Data models and schemas
    - `database/` - SQLAlchemy models and repositories
    - `llm/` - LLM factory and providers
    - `schemas/` - Pydantic schemas
  - `services/` - Business logic layer
  - `tasks/` - Celery background tasks
    - `background/` - Task implementations (PDF, embeddings, cleanup)
  - `utils/` - Utility functions

### Key Services
- **Authentication**: JWT-based auth with role-based access
- **Chat**: Streaming responses with multiple LLM model support
- **File Processing**: PDF/TXT/DOCX upload with vectorization
- **RAG**: Document retrieval and question answering
- **Caching**: Redis for performance optimization
- **Background Tasks**: Celery for async processing

## Testing Strategy

### Backend Tests
- **Unit tests**: `tests/unit/` - Fast, isolated tests
- **Integration tests**: `tests/integration/` - External service integration
- **E2E tests**: `tests/e2e/` - Complete workflow testing
- **Test markers**: Use pytest markers (`@pytest.mark.unit`, `@pytest.mark.integration`)

### Test Execution
```bash
# Run specific test types
poetry run pytest -m unit
poetry run pytest -m integration
poetry run pytest -m e2e

# Run tests with specific patterns
poetry run pytest tests/unit/test_services/
poetry run pytest tests/integration/test_api/test_auth_routes.py

# Debug failing tests
poetry run pytest -x --pdb  # Drop into debugger on failure
```

## Development Workflow

1. **Start backend services**:
   ```bash
   # Terminal 1: Backend API
   cd langchain_backend && poetry run python -m app.main
   
   # Terminal 2: Redis
   redis-server
   
   # Terminal 3: Celery worker
   cd langchain_backend && poetry run python start_celery_worker.py
   
   # Terminal 4: Flower monitoring (optional)
   cd langchain_backend && poetry run python start_flower.py
   ```

2. **Start frontend**:
   ```bash
   cd bolt-chatbot-ui && npm run dev
   ```

3. **Access services**:
   - Frontend: http://localhost:3000
   - Backend API: http://localhost:8000
   - API Docs: http://localhost:8000/docs
   - Flower: http://localhost:5555

## Environment Configuration

### Backend (.env)
```bash
# Database
DATABASE_URL=sqlite+aiosqlite:///./db/app.db

# Redis
REDIS_URL=redis://localhost:6379/1

# Security
JWT_SECRET=your_super_secret_jwt_key
JWT_ALGORITHM=HS256

# LLM Providers
OPENAI_API_KEY=your_openai_api_key
TAVILY_API_KEY=your_tavily_api_key

# Environment
ENVIRONMENT=development
```

### Frontend Environment
Environment variables are configured in `lib/api.ts` and through Next.js runtime configuration.

## Common Development Tasks

### Adding New API Endpoints
1. Create schema in `app/models/schemas/`
2. Add route in `app/api/routes/`
3. Implement service in `app/services/`
4. Add tests in appropriate test directory

### Adding New UI Components
1. Create component in `components/`
2. Add to Storybook if needed
3. Import and use in pages

### Background Tasks
- Tasks are defined in `app/tasks/background/`
- Use `@app.tasks.core.task_decorators.background_task` decorator
- Tasks are automatically registered with Celery

## Debugging Tips

### Backend Debugging
```bash
# Enable debug logging
export LOG_LEVEL=DEBUG

# Run with reload for development
poetry run python -m app.main --reload

# Debug specific test
poetry run pytest tests/unit/test_services/test_auth_service.py -v --pdb
```

### Frontend Debugging
```bash
# Build with verbose output
npm run build --verbose

# Check TypeScript errors
npx tsc --noEmit
```

## Deployment

### Docker Deployment
```bash
# Build and start
cd bolt-chatbot-ui
docker-compose up -d --build

# View logs
docker-compose logs -f
```

### Kubernetes Deployment
```bash
# Using Helm
cd bolt-chatbot-ui/chatbot-helm
helm install chat-bot . -f values.yaml

# Using kubectl
kubectl apply -f k8s.yaml
```

## Monitoring & Observability

- **Flower**: Celery task monitoring at http://localhost:5555
- **Health checks**: `GET /api/v1/health`
- **Logging**: Structured JSON logging with different levels
- **Metrics**: Built-in Prometheus metrics (if configured)

## API Client Generation

The backend supports OpenAPI specification generation:
```bash
cd langchain_backend
./scripts/export_api.sh  # Generates openapi.json and openapi.yaml
```

Frontend uses type-safe API client in `lib/api.ts` with auto-generated types.