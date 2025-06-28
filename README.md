# Chat Bot - AI Chatbot Full-Stack Project

A modern AI chatbot full-stack application featuring a Next.js frontend UI and a FastAPI + LangChain backend API. Supports real-time streaming conversations, file upload processing, RAG Q&A systems, and more.

## 🚀 Features

### Frontend Features (Next.js)
- **Modern UI Interface**: Built with React 19 + Next.js 15, using Tailwind CSS + Radix UI
- **User Authentication System**: JWT login/registration with user state management
- **Real-time Chat Interface**: Streaming message display with multi-conversation management
- **File Upload Functionality**: Drag-and-drop upload with file preview and management
- **Responsive Design**: Desktop and mobile responsive design
- **Dark/Light Theme**: Automatic theme switching support

### Backend Features (FastAPI + LangChain)
- **Modern FastAPI Architecture**: Async support with SQLAlchemy 2.0 database operations
- **Authentication & Authorization**: JWT authentication with role-based access control
- **Intelligent Chat System**: Multiple LLM model integration with streaming responses
- **RAG Enhancement**: Document vectorization storage with ChromaDB vector database
- **Background Task Processing**: Celery async tasks for file processing and vectorization
- **Cache Optimization**: Redis caching for improved performance
- **Monitoring & Logging**: Health checks and Flower task monitoring
- **Automatic API Documentation**: OpenAPI specification with auto-generated client code

## 📁 Project Structure

```
chat-bot/
├── bolt-chatbot-ui/          # Next.js Frontend Project
│   ├── app/                  # Next.js App Router
│   │   ├── (auth)/          # Authentication Pages
│   │   ├── api/             # API Routes
│   │   └── page.tsx         # Main Chat Page
│   ├── components/          # React Components
│   │   ├── auth/           # Authentication Components
│   │   ├── ui/             # Base UI Components
│   │   ├── ChatHistory.tsx # Chat History
│   │   ├── ChatInput.tsx   # Message Input
│   │   └── ChatMessage.tsx # Message Display
│   ├── lib/                # Utility Libraries and API Client
│   ├── config/             # Configuration Files
│   └── hooks/              # React Hooks
│
└── langchain_backend/        # FastAPI Backend Project
    ├── app/                  # Main Application
    │   ├── api/             # API Routes and Middleware
    │   ├── chains/          # LangChain Implementation
    │   ├── models/          # Data Models and Schemas
    │   ├── services/        # Business Logic Layer
    │   ├── tasks/           # Celery Background Tasks
    │   └── utils/           # Utility Functions
    ├── tests/               # Test Suites
    ├── scripts/             # Development Scripts
    └── docs/                # Project Documentation
```

## 🔧 Quick Start

### Prerequisites

- **Node.js**: 18.0+ (Frontend)
- **Python**: 3.9+ (Backend)
- **Redis**: 6.0+ (Cache and Task Queue)
- **PostgreSQL**: 13+ (Production, SQLite for Development)

### Installation

#### 1. Clone the Repository

```bash
git clone <repository-url>
cd chat-bot
```

#### 2. Backend Setup

```bash
# Enter backend directory
cd langchain_backend

# Install Python dependencies (Poetry recommended)
poetry install
# or use pip
pip install -r requirements.txt

# Set up environment variables
cp .env.example .env
# Edit .env file, configure database, API keys, etc.

# Initialize database
poetry run alembic upgrade head

# Start backend service
poetry run python -m app.main
```

Backend service will start at `http://localhost:8000`

#### 3. Frontend Setup

```bash
# Enter frontend directory
cd bolt-chatbot-ui

# Install Node.js dependencies
npm install
# or use yarn
yarn install

# Start development server
npm run dev
```

Frontend application will start at `http://localhost:3000`

#### 4. Start Background Services (Optional but Recommended)

```bash
# Start Redis (new terminal)
redis-server

# Start Celery worker (new terminal)
cd langchain_backend
poetry run python start_celery_worker.py

# Start Flower monitoring (new terminal)
poetry run python start_flower.py
```

## 🌐 Service Access URLs

- **Frontend Application**: http://localhost:3000
- **Backend API**: http://localhost:8000
- **API Documentation**: http://localhost:8000/docs (Swagger UI)
- **ReDoc Documentation**: http://localhost:8000/redoc
- **Flower Monitoring**: http://localhost:5555 (if Celery is started)

## 📖 Usage Guide

### User Authentication
1. Visit the frontend application homepage
2. Click register to create a new account or login with existing account
3. Successfully login to enter the chat interface

### Chat Features
1. Enter messages in the chat input box
2. Select AI model (default: Gemma2)
3. Click send or press Enter key
4. Support creating multiple conversations and switching chat history

### File Upload
1. Click the file upload button or drag files to the designated area
2. Support multiple formats: PDF, TXT, DOC, DOCX
3. Files will be automatically processed and vectorized for storage
4. In RAG mode, you can perform Q&A based on uploaded documents

### RAG Retrieval Q&A
1. Upload relevant document files
2. Select documents to use in the document panel
3. Ask questions related to document content
4. AI will provide accurate answers based on document content

## 🛠️ Technology Stack

### Frontend Stack
- **Framework**: Next.js 15.3.4 + React 19.1.0
- **Language**: TypeScript
- **Styling**: Tailwind CSS
- **Component Library**: Radix UI
- **State Management**: React Hooks + Context
- **HTTP Client**: Axios + Hey-API Client
- **Form Handling**: React Hook Form + Zod
- **Theming**: next-themes

### Backend Stack
- **Framework**: FastAPI
- **Language**: Python 3.9+
- **AI Framework**: LangChain
- **Database**: SQLAlchemy 2.0 + PostgreSQL/SQLite
- **Caching**: Redis
- **Task Queue**: Celery
- **Vector Database**: ChromaDB
- **Authentication**: JWT (PyJWT)
- **API Documentation**: OpenAPI (Swagger)

### DevOps & Deployment
- **Containerization**: Docker + Docker Compose
- **Orchestration**: Kubernetes (Helm Charts)
- **Reverse Proxy**: Nginx
- **Monitoring**: Flower (Celery), Health Checks
- **CI/CD**: GitHub Actions

## 🧪 Testing

### Backend Tests
```bash
cd langchain_backend

# Run all tests
poetry run pytest

# Run tests with coverage
poetry run pytest --cov=app

# Run specific test types
poetry run pytest tests/unit/      # Unit tests
poetry run pytest tests/integration/  # Integration tests
poetry run pytest tests/e2e/      # End-to-end tests
```

### Frontend Tests
```bash
cd bolt-chatbot-ui

# Run tests (if configured)
npm run test

# Type checking
npm run type-check

# Linting
npm run lint
```

## 🚀 Deployment

### Docker Deployment

```bash
# Build and start all services
docker-compose up -d

# Check service status
docker-compose ps

# View logs
docker-compose logs -f
```

### Kubernetes Deployment

```bash
# Deploy using Helm
cd bolt-chatbot-ui/chatbot-helm
helm install chat-bot . -f values.yaml

# Or use K8s configuration directly
kubectl apply -f k8s.yaml
```

### Production Environment Configuration

#### Environment Variables
```bash
# Database configuration
DATABASE_URL=postgresql://username:password@localhost:5432/chatbot_db

# API keys
OPENAI_API_KEY=your_openai_api_key
TAVILY_API_KEY=your_tavily_api_key  # Optional

# Security configuration
JWT_SECRET=your_super_secret_jwt_key
JWT_ALGORITHM=HS256

# Redis configuration
REDIS_URL=redis://localhost:6379

# File storage
STORAGE_TYPE=s3  # local, s3, minio
MAX_FILE_SIZE=52428800  # 50MB

# Vector storage
VECTOR_STORE_TYPE=chroma
CHROMA_PERSIST_DIR=./chroma_db
```

## 🔄 API Integration

### Auto-Generate Frontend Client

The backend project supports automatic OpenAPI specification export and type-safe frontend client generation:

```bash
cd langchain_backend

# Export API specification
./scripts/export_api.sh

# Generated files are located in ./api-spec/
# - openapi.json (JSON format)
# - openapi.yaml (YAML format)
```

### Frontend API Client Usage

The project has integrated type-safe API client:

```typescript
// In lib/api.ts
import { api } from './api-client';

// Type-safe API calls
const user = await getCurrentUser();
const conversations = await getConversations();
const response = await sendChatMessage(conversationId, message);
```

## 📚 Documentation

- [Backend API Documentation](./langchain_backend/docs/)
- [Architecture Design Documentation](./langchain_backend/docs/ARCHITECTURE_PROPOSAL.md)
- [API Client Generation Guide](./langchain_backend/docs/API_CLIENT_GENERATION.md)
- [Deployment Guide](./langchain_backend/docs/deployment.md)

## 🤝 Contributing

1. Fork this project
2. Create a feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Create a Pull Request

### Development Guidelines
- Follow TypeScript/Python coding standards
- Add test cases for new features
- Ensure all tests pass
- Update relevant documentation

## 🐛 Issues & Feedback

If you encounter any problems or have suggestions for improvement:

- Submit issue reports in GitHub Issues
- Check existing Issues and solutions
- Refer to the FAQ in project documentation

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details

## 🙏 Acknowledgments

- [LangChain](https://github.com/langchain-ai/langchain) - AI application framework
- [FastAPI](https://github.com/tiangolo/fastapi) - Modern Python web framework
- [Next.js](https://nextjs.org/) - React full-stack framework
- [Radix UI](https://www.radix-ui.com/) - Low-level UI primitives
- [Tailwind CSS](https://tailwindcss.com/) - Utility-first CSS framework

## 📞 Contact

- Project Maintainer: [Your Name](mailto:your.email@example.com)
- Project Homepage: [GitHub Repository](https://github.com/your-username/chat-bot)
- Issue Reports: [GitHub Issues](https://github.com/your-username/chat-bot/issues)

---

⭐ If this project helps you, please give us a Star! 