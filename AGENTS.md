# Nexora AI - Architecture & Development Rules

## Project Overview

Nexora AI is a production-ready conversational AI platform designed as **Version 1** of a larger future media ecosystem (Nexora).

### Current Scope (V1)
- AI-powered conversational assistant
- Web search integration for current information
- Image generation foundation (extensible)
- Video generation job system (extensible)
- User authentication and profiles
- Conversation history and management

### Future Scope (V2+)
- Video uploads and processing
- Creator channels and profiles
- Subscriptions and following
- Likes, comments, and engagement
- AI-generated video/thumbnails
- Live streaming
- Creator analytics and monetization

## Architecture

### Tech Stack

```
Frontend:
  - Next.js 14+ (App Router)
  - TypeScript
  - Tailwind CSS
  - React Query / SWR
  - UI: Responsive dark/light theme

Backend:
  - FastAPI (Python 3.11+)
  - SQLAlchemy ORM
  - Alembic migrations
  - JWT + session-based auth

Database:
  - PostgreSQL 14+
  - SQLAlchemy models
  - Alembic version control

Storage:
  - Local filesystem (dev)
  - S3-compatible (prod)

External Integrations:
  - AI Provider (OpenAI, Anthropic, etc.) - abstracted
  - Search Provider (Google, Bing, etc.) - abstracted
  - Image Provider (DALL-E, Midjourney, etc.) - abstracted
  - Video Provider (future - Runway, Pika, etc.) - abstracted
```

### Directory Structure

```
nexora-ai/
├── frontend/                 # Next.js application
│   ├── src/
│   │   ├── app/             # App Router pages
│   │   ├── components/      # React components
│   │   ├── lib/             # Utilities, hooks, API clients
│   │   ├── types/           # TypeScript types
│   │   └── styles/          # Global styles
│   ├── public/              # Static assets
│   ├── package.json
│   └── tsconfig.json
├── backend/                  # FastAPI application
│   ├── app/
│   │   ├── main.py          # Application entry
│   │   ├── config.py        # Configuration & env
│   │   ├── security.py      # Auth & passwords
│   │   ├── middleware.py    # Middleware (CORS, etc)
│   │   ├── models/          # SQLAlchemy models
│   │   ├── schemas/         # Pydantic schemas
│   │   ├── routers/         # API routes
│   │   ├── services/        # Business logic
│   │   ├── providers/       # AI/Search/Image/Video abstraction
│   │   ├── storage/         # Storage abstraction
│   │   ├── database.py      # DB connection
│   │   └── dependencies.py  # Dependency injection
│   ├── alembic/             # Database migrations
│   ├── tests/               # Unit & integration tests
│   ├── requirements.txt
│   └── .env.example
├── docker-compose.yml       # Dev environment
├── README.md
├── AGENTS.md               # This file
└── .env.example            # Environment variables template
```

## Core Principles

### 1. Provider Abstraction
**Never hard-code external integrations.**

- All AI, search, image, and video services use abstract base classes
- Providers are configured via environment variables
- Multiple implementations available (real + stubs)
- Invalid/unconfigured providers return clear error messages
- No fake generation results

Example:
```python
class AIProvider(ABC):
    @abstractmethod
    async def generate_response(self, prompt: str, history: List) -> str: ...
    
    @abstractmethod
    async def stream_response(self, prompt: str, history: List) -> AsyncIterator[str]: ...

class OpenAIProvider(AIProvider): ...
class StubAIProvider(AIProvider):
    async def generate_response(...):
        raise NotConfiguredError("AI provider not configured")
```

### 2. No Secrets in Code
- All API keys, database URLs, and credentials in environment variables
- `.env.example` documents all required variables
- `.env` never committed
- Production uses environment-based configuration

### 3. Separation of Concerns
- **Models**: Database schema (SQLAlchemy)
- **Schemas**: Request/response validation (Pydantic)
- **Services**: Business logic (reusable)
- **Routers**: HTTP endpoints
- **Providers**: External integrations (abstracted)
- **Storage**: File/media persistence (abstracted)

### 4. Authentication
- Passwords: hashed with bcrypt
- Tokens: JWT with short TTL
- Sessions: optional persistent sessions
- Authorization: role-based (user, admin, creator for future)
- Rate limiting: per-user and global limits

### 5. Database Design
- Users: auth, profile, settings
- Conversations: user-owned chat sessions
- Messages: individual chat messages with metadata
- AIRequests: tracking of AI/search/generation requests
- Media: images, videos, and generated content
- GenerationJobs: async video/image generation with status
- SearchResults: cached search queries with sources

### 6. Error Handling
- Structured error responses (error_code, message, details)
- Validation errors: 422 with field details
- Auth errors: 401/403 with safe messages
- Server errors: 500 with generic message (log details)
- Client errors: clear, actionable messages
- No API keys or sensitive data in error messages

### 7. Security
- Input validation: Pydantic schemas
- SQL injection: SQLAlchemy parameterized queries
- CORS: configured correctly
- CSRF: SameSite cookies
- Rate limiting: token bucket per user
- File uploads: type and size validation
- Content Security Policy headers
- HTTPS in production

### 8. Testing
- Unit tests for services and providers
- Integration tests for API endpoints
- Auth and authorization tests
- Database migration tests
- At least 70% code coverage target

## Coding Standards

### Python (Backend)
- Type hints: all functions and variables
- Docstrings: Google style
- PEP 8: use black and flake8
- Async/await: for I/O-bound operations
- No print(): use logging

### TypeScript (Frontend)
- Type hints: strict mode
- Components: functional + hooks
- No `any`: use proper types
- Error handling: try/catch with user feedback
- API calls: centralized in /lib

## Data Models (Initial)

### User
```python
- id: UUID
- email: str (unique, lowercase)
- username: str (unique)
- password_hash: str
- full_name: str
- avatar_url: str (optional)
- bio: str (optional)
- language: str (default: 'en')
- theme: str (default: 'dark')
- created_at: datetime
- updated_at: datetime
- is_active: bool
- is_verified: bool
```

### Conversation
```python
- id: UUID
- user_id: UUID (foreign key)
- title: str
- description: str (optional)
- language: str (default: 'en')
- created_at: datetime
- updated_at: datetime
- is_archived: bool
- metadata: JSON (future: model used, settings, etc)
```

### Message
```python
- id: UUID
- conversation_id: UUID (foreign key)
- user_id: UUID (foreign key)
- role: enum (user, assistant)
- content: str
- is_markdown: bool
- created_at: datetime
- metadata: JSON
```

### AIRequest
```python
- id: UUID
- user_id: UUID (foreign key)
- conversation_id: UUID (foreign key, optional)
- request_type: enum (chat, image, video, search)
- provider: str
- prompt: str
- tokens_used: int (optional)
- response_status: enum (pending, success, failed)
- error_message: str (optional)
- created_at: datetime
- completed_at: datetime (optional)
```

### Media
```python
- id: UUID
- user_id: UUID (foreign key)
- ai_request_id: UUID (foreign key, optional)
- media_type: enum (image, video)
- storage_path: str
- storage_provider: str
- url: str
- metadata: JSON
- created_at: datetime
- size_bytes: int
```

### GenerationJob
```python
- id: UUID
- user_id: UUID (foreign key)
- ai_request_id: UUID (foreign key)
- job_type: enum (image, video)
- provider: str
- prompt: str
- status: enum (queued, processing, completed, failed, cancelled)
- progress_percent: int (0-100)
- result_media_id: UUID (foreign key, optional)
- error_message: str (optional)
- created_at: datetime
- started_at: datetime (optional)
- completed_at: datetime (optional)
- estimated_completion_at: datetime (optional)
```

### SearchResult
```python
- id: UUID
- query: str (hashed for grouping)
- title: str
- url: str
- snippet: str
- source: str
- cached_at: datetime
- expires_at: datetime
```

## API Endpoints (V1)

### Authentication
- POST `/api/auth/register` - Register user
- POST `/api/auth/login` - Login
- POST `/api/auth/logout` - Logout
- GET `/api/auth/me` - Current user
- POST `/api/auth/refresh` - Refresh token
- POST `/api/auth/verify-email` - Email verification

### User
- GET `/api/users/{user_id}` - Get user profile
- PUT `/api/users/{user_id}` - Update profile
- PUT `/api/users/{user_id}/settings` - Update settings
- DELETE `/api/users/{user_id}` - Delete account

### Conversations
- GET `/api/conversations` - List conversations
- POST `/api/conversations` - Create conversation
- GET `/api/conversations/{conv_id}` - Get conversation
- PUT `/api/conversations/{conv_id}` - Rename conversation
- DELETE `/api/conversations/{conv_id}` - Delete conversation

### Messages
- GET `/api/conversations/{conv_id}/messages` - Get messages
- POST `/api/conversations/{conv_id}/messages` - Send message
- GET `/api/conversations/{conv_id}/messages/{msg_id}` - Get message
- DELETE `/api/conversations/{conv_id}/messages/{msg_id}` - Delete message

### Chat (Streaming)
- POST `/api/chat/stream` - Stream AI response
- POST `/api/chat/regenerate` - Regenerate last response
- POST `/api/chat/stop` - Stop generation

### Search
- GET `/api/search?q=...` - Web search

### Image Generation
- POST `/api/images/generate` - Request image
- GET `/api/images/{image_id}` - Get image
- GET `/api/images/{image_id}/status` - Get generation status

### Video Generation
- POST `/api/videos/generate` - Request video
- GET `/api/videos/{job_id}` - Get job
- GET `/api/videos/{job_id}/status` - Get status
- POST `/api/videos/{job_id}/cancel` - Cancel job

## Environment Variables

See `.env.example` for the complete list.

Key categories:
- Database: `DATABASE_URL`
- AI: `AI_PROVIDER`, `AI_API_KEY`, `AI_MODEL`
- Search: `SEARCH_PROVIDER`, `SEARCH_API_KEY`
- Image: `IMAGE_PROVIDER`, `IMAGE_API_KEY`
- Video: `VIDEO_PROVIDER`, `VIDEO_API_KEY`
- Storage: `STORAGE_TYPE`, `S3_*` (for production)
- Auth: `SECRET_KEY`, `JWT_ALGORITHM`, `JWT_EXPIRY`
- CORS: `ALLOWED_ORIGINS`

## Development Workflow

### Setup
```bash
# Backend
cd backend
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
alembic upgrade head
python -m pytest tests/

# Frontend
cd ../frontend
npm install
npm run dev
```

### Running
```bash
# Terminal 1: PostgreSQL
docker-compose up -d postgres

# Terminal 2: FastAPI backend
cd backend
source venv/bin/activate
uvicorn app.main:app --reload --port 8000

# Terminal 3: Next.js frontend
cd frontend
npm run dev
# Open http://localhost:3000
```

### Database Migrations
```bash
cd backend
# Auto-generate migration
alembic revision --autogenerate -m "Add users table"
# Apply migrations
alembic upgrade head
# Downgrade
alembic downgrade -1
```

## Testing

```bash
cd backend

# Run all tests
pytest

# With coverage
pytest --cov=app tests/

# Specific test file
pytest tests/test_auth.py

# Watch mode
pytest-watch
```

## Deployment Checklist (Future)

- [ ] Environment variables set in production
- [ ] Database backups configured
- [ ] Rate limiting enforced
- [ ] HTTPS/TLS enabled
- [ ] CORS properly restricted
- [ ] API keys rotated
- [ ] Security headers set
- [ ] Logging and monitoring configured
- [ ] Error tracking (Sentry) enabled
- [ ] Database connection pooling
- [ ] CDN for static files
- [ ] S3 configured for media storage
- [ ] Email service configured
- [ ] Backup storage location configured

## Future Considerations (V2+)

For video platform features:
- Add `Creator` role with channel data
- Add `Video` model with upload, processing, thumbnails
- Add `Playlist` model
- Add `Like`, `Comment`, `Subscription` models
- Add `Watch` model for history
- Add `Recommendation` service
- Add `LiveStream` model and WebRTC integration
- Add `Analytics` service for creator dashboard
- Add `Copyright` and `Report` systems
- Storage will be heavily used: plan for CDN and multi-region

These are not implemented in V1 but the database and API structure supports adding them.

## Questions & Support

For architecture decisions or coding standards questions, refer to this document first.

---

Last updated: 2026-09-30
