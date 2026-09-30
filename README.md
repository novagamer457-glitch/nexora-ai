# Nexora AI

Production-ready AI conversational platform with extensible architecture for future media capabilities.

## Features (Version 1)

- ✅ Modern AI chat interface (English & Hindi support)
- ✅ Conversation history and management
- ✅ Streaming AI responses
- ✅ Web search for current information with source citations
- ✅ Image generation foundation (extensible)
- ✅ Video generation job system (extensible)
- ✅ User authentication and profiles
- ✅ Secure password hashing with bcrypt
- ✅ JWT + session-based authentication
- ✅ Rate limiting
- ✅ Markdown and code block rendering
- ✅ Copy/regenerate/stop response controls
- ✅ Error handling and input validation
- ✅ Dark/light theme support
- ✅ Responsive mobile design

## Architecture

### Tech Stack

**Frontend:**
- Next.js 14+ (App Router)
- TypeScript
- Tailwind CSS
- Responsive design

**Backend:**
- FastAPI (Python 3.11+)
- SQLAlchemy ORM
- Alembic migrations
- PostgreSQL 14+
- JWT authentication

**Storage:**
- Local filesystem (development)
- S3-compatible (production)

**Providers (Abstracted):**
- AI chat provider (OpenAI, Anthropic, etc.)
- Web search provider (Google, Bing, etc.)
- Image generation provider (DALL-E, Midjourney, etc.)
- Video generation provider (future: Runway, Pika, etc.)

### Project Structure

```
nexora-ai/
├── frontend/                 # Next.js application
│   ├── src/
│   │   ├── app/             # App Router pages
│   │   ├── components/      # React components
│   │   ├── lib/             # API clients, hooks, utilities
│   │   ├── types/           # TypeScript types
│   │   └── styles/          # CSS
│   ├── public/
│   ├── package.json
│   └── tsconfig.json
│
├── backend/                  # FastAPI application
│   ├── app/
│   │   ├── main.py          # Entry point
│   │   ├── config.py        # Configuration
│   │   ├── security.py      # Auth & hashing
│   │   ├── middleware.py    # CORS, rate limiting
│   │   ├── models/          # SQLAlchemy models
│   │   ├── schemas/         # Pydantic schemas
│   │   ├── routers/         # API routes
│   │   ├── services/        # Business logic
│   │   ├── providers/       # AI/Search/Image/Video
│   │   ├── storage/         # File storage
│   │   ├── database.py      # DB connection
│   │   └── dependencies.py  # DI
│   ├── alembic/             # Migrations
│   ├── tests/               # Tests
│   ├── requirements.txt
│   └── .env.example
│
├── README.md
├── AGENTS.md                # Architecture document
├── docker-compose.yml
└── .env.example
```

## Quick Start

### Prerequisites

- Python 3.11+
- Node.js 18+
- PostgreSQL 14+ (or use Docker)
- Docker Compose (optional)

### Installation

**1. Clone the repository:**

```bash
git clone https://github.com/novagamer457-glitch/nexora-ai.git
cd nexora-ai
```

**2. Backend Setup:**

```bash
cd backend
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

**3. Database Setup:**

```bash
# Start PostgreSQL with Docker
docker-compose up -d postgres

# Run migrations
cd backend
alembic upgrade head
```

**4. Frontend Setup:**

```bash
cd frontend
npm install
```

**5. Configuration:**

```bash
# Copy environment template
cp .env.example .env

# Edit .env with your API keys and configuration
# Minimum required:
# - DATABASE_URL
# - SECRET_KEY
# - AI_PROVIDER and AI_API_KEY
```

### Running Locally

**Terminal 1 - Database:**

```bash
docker-compose up postgres
```

**Terminal 2 - Backend:**

```bash
cd backend
source venv/bin/activate
uvicorn app.main:app --reload --port 8000
# Backend: http://localhost:8000
# API docs: http://localhost:8000/docs
```

**Terminal 3 - Frontend:**

```bash
cd frontend
npm run dev
# Frontend: http://localhost:3000
```

### Testing

```bash
cd backend

# Run all tests
pytest

# Run with coverage
pytest --cov=app tests/

# Run specific test file
pytest tests/test_auth.py -v
```

## API Documentation

Full interactive API documentation available at:
- **Swagger UI:** http://localhost:8000/docs
- **ReDoc:** http://localhost:8000/redoc

### Core Endpoints

**Authentication:**
- `POST /api/auth/register` - Register new user
- `POST /api/auth/login` - Login user
- `POST /api/auth/logout` - Logout
- `GET /api/auth/me` - Get current user
- `POST /api/auth/refresh` - Refresh token

**Conversations:**
- `GET /api/conversations` - List all conversations
- `POST /api/conversations` - Create conversation
- `GET /api/conversations/{id}` - Get conversation
- `PUT /api/conversations/{id}` - Rename conversation
- `DELETE /api/conversations/{id}` - Delete conversation

**Messages:**
- `POST /api/conversations/{conv_id}/messages` - Send message
- `GET /api/conversations/{conv_id}/messages` - Get messages
- `DELETE /api/conversations/{conv_id}/messages/{msg_id}` - Delete message

**Chat (Streaming):**
- `POST /api/chat/stream` - Stream AI response
- `POST /api/chat/regenerate` - Regenerate last response
- `POST /api/chat/stop` - Stop generation

**Search:**
- `GET /api/search?q=...` - Web search

**Image Generation:**
- `POST /api/images/generate` - Request image generation
- `GET /api/images/{image_id}` - Get image
- `GET /api/images/{image_id}/status` - Get generation status

**Video Generation:**
- `POST /api/videos/generate` - Request video generation
- `GET /api/videos/{job_id}` - Get job details
- `GET /api/videos/{job_id}/status` - Get job status
- `POST /api/videos/{job_id}/cancel` - Cancel job

**User:**
- `GET /api/users/{user_id}` - Get user profile
- `PUT /api/users/{user_id}` - Update profile
- `PUT /api/users/{user_id}/settings` - Update settings

## Environment Variables

See `.env.example` for the complete list. Key variables:

| Variable | Description | Required |
|----------|-------------|----------|
| `DATABASE_URL` | PostgreSQL connection string | Yes |
| `SECRET_KEY` | JWT secret (min 32 chars) | Yes |
| `AI_PROVIDER` | AI provider (openai, anthropic) | Yes |
| `AI_API_KEY` | API key for AI provider | Yes* |
| `AI_MODEL` | Model name (e.g., gpt-4) | Yes |
| `SEARCH_PROVIDER` | Search provider (google, bing) | No |
| `SEARCH_API_KEY` | API key for search provider | No* |
| `IMAGE_PROVIDER` | Image provider (openai, etc.) | No |
| `IMAGE_API_KEY` | API key for image provider | No* |
| `STORAGE_TYPE` | Storage type (local or s3) | No |
| `ALLOWED_ORIGINS` | CORS allowed origins | No |

*\* Required if provider is configured*

## Database Models

- **User** - Authentication, profile, settings
- **Conversation** - Chat session
- **Message** - Individual chat message
- **AIRequest** - Tracks all AI/search/generation requests
- **Media** - Images, videos, generated content
- **GenerationJob** - Async image/video generation with status tracking
- **SearchResult** - Cached search queries with sources
- **UserSettings** - User preferences and configuration

## Development

### Database Migrations

```bash
cd backend

# Create new migration
alembic revision --autogenerate -m "Description"

# Apply all pending migrations
alembic upgrade head

# Downgrade one migration
alembic downgrade -1

# View migration history
alembic history
```

### Code Style

**Backend:** Black, Flake8, Type hints

```bash
cd backend
black app tests
flake8 app tests
mypy app
```

**Frontend:** ESLint, Prettier

```bash
cd frontend
npm run lint
npm run format
```

## Security Features

- ✅ Secure password hashing (bcrypt)
- ✅ JWT token-based authentication
- ✅ Input validation (Pydantic)
- ✅ SQL injection prevention (SQLAlchemy ORM)
- ✅ CORS configuration
- ✅ Rate limiting (token bucket)
- ✅ File type and size validation
- ✅ API key protection (environment variables)
- ✅ Structured error responses (no sensitive data)
- ✅ Security headers (HTTPS ready)

## Testing

Tests cover:
- ✅ Authentication (register, login, JWT)
- ✅ Authorization (role-based access)
- ✅ Chat service (message creation, history)
- ✅ Providers (AI, search, image, video)
- ✅ Database models and migrations
- ✅ Rate limiting
- ✅ Input validation

Run tests:

```bash
cd backend
pytest tests/ -v --cov=app
```

## Future Roadmap (Version 2+)

**Planned for future versions:**
- Video uploads and processing
- Creator channels and profiles
- Subscriptions and following
- Likes, comments, reactions
- Playlists and watch history
- Search and recommendations
- Live streaming
- AI-generated videos and thumbnails
- Creator dashboard and analytics
- Monetization and revenue sharing

The current architecture supports adding these features without major changes to core systems.

## Troubleshooting

**Backend won't start:**
- Check Python version (3.11+): `python --version`
- Verify PostgreSQL is running: `docker-compose ps`
- Check `DATABASE_URL` in `.env`
- Run migrations: `alembic upgrade head`
- Check logs: `uvicorn app.main:app --reload`

**Frontend API errors:**
- Verify backend is running on `http://localhost:8000`
- Check `NEXT_PUBLIC_API_URL` in frontend `.env`
- Check CORS settings: see backend logs
- Clear browser cache: Ctrl+Shift+Delete

**Database errors:**
- Ensure PostgreSQL is running: `docker-compose ps`
- Check connection string in `.env`
- Run migrations: `alembic upgrade head`
- Check postgres logs: `docker-compose logs postgres`

**AI responses not working:**
- Verify `AI_PROVIDER` is set in `.env`
- Check `AI_API_KEY` is valid
- Review backend logs for provider errors
- Test with curl: `curl -X GET http://localhost:8000/docs`

**Tests failing:**
- Ensure PostgreSQL is running
- Set `DATABASE_URL` or use default test DB
- Run migrations: `alembic upgrade head`
- Clear pytest cache: `pytest --cache-clear`

## Contributing

1. Read architecture guidelines in `AGENTS.md`
2. Create feature branch: `git checkout -b feature/your-feature`
3. Make changes following code style
4. Add tests for new features
5. Run tests: `pytest tests/`
6. Submit PR against `main`

## License

MIT

## Support

- 📚 **Documentation:** See `AGENTS.md`
- 🐛 **Issues:** GitHub Issues
- 💬 **Discussions:** GitHub Discussions
- 📖 **API Docs:** http://localhost:8000/docs

---

**Status:** Version 1.0 - Initial Release
**Last Updated:** 2026-09-30
