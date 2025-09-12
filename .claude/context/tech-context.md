---
created: 2025-09-12T05:26:33Z
last_updated: 2025-09-12T05:26:33Z
version: 1.0
author: Claude Code PM System
---

# Technical Context

## Technology Stack Overview

ShareStack adopts a modern, scalable full-stack architecture designed for high-performance knowledge sharing and monetization platform.

## Frontend Technologies

### PC Web Application
- **Framework**: Next.js 14+ with App Router
- **UI Library**: shadcn/ui + Tailwind CSS 3.0+
- **State Management**: Zustand (lightweight, TypeScript-first)
- **Data Fetching**: TanStack Query (React Query v5)
- **Authentication**: NextAuth.js or JWT-based custom solution
- **Form Handling**: React Hook Form + Zod validation
- **Rich Text**: Markdown editor with syntax highlighting
- **Build Tool**: Built-in Next.js with Turbopack
- **Language**: TypeScript 5.0+

### Mobile Application
- **Framework**: React Native + Expo SDK 50+
- **UI Components**: NativeBase or Tamagui
- **Navigation**: React Navigation 6
- **State Management**: Zustand (consistent with web)
- **Networking**: Axios + TanStack Query (React Query)
- **Authentication**: Expo AuthSession + secure storage
- **Platform**: Cross-platform (iOS + Android)

## Backend Technologies

### API Server
- **Framework**: FastAPI 3.11+ (Python)
- **ORM**: SQLAlchemy 2.0 with async support
- **Database Migration**: Alembic
- **Validation**: Pydantic v2 models
- **Authentication**: JWT tokens + OAuth2
- **API Documentation**: Automatic OpenAPI/Swagger generation
- **ASGI Server**: Uvicorn with Gunicorn for production

### Task Processing
- **Message Broker**: Redis 7
- **Task Queue**: Celery with Redis backend
- **Scheduling**: Celery Beat for periodic tasks
- **Background Jobs**: Payment processing, email sending, analytics

## Database & Storage

### Primary Database
- **RDBMS**: MySQL 8.0
- **Features**: Full-text search, JSON columns, proper indexing
- **Connection Pool**: SQLAlchemy async pool
- **Migration Strategy**: Alembic for schema versioning

### Cache & Session Storage
- **Cache**: Redis 7 for application cache
- **Session Store**: Redis for user sessions
- **Real-time**: Redis for WebSocket/SSE support

### File Storage
- **Object Storage**: AWS S3 (primary) or 阿里云OSS (China)
- **CDN**: CloudFlare for global content delivery
- **Media Processing**: On-demand image resizing, video transcoding

## Infrastructure & DevOps

### Containerization
- **Container Runtime**: Docker
- **Development**: Docker Compose for local development
- **Production**: Kubernetes or Docker Swarm

### Web Server
- **Reverse Proxy**: Nginx
- **Load Balancer**: AWS ALB or Nginx upstream
- **SSL/TLS**: Let's Encrypt with automatic renewal

### Monitoring & Observability
- **Metrics**: Prometheus + Grafana
- **Logging**: Structured logging with JSON format
- **Error Tracking**: Sentry for error monitoring
- **Health Checks**: Built-in FastAPI health endpoints

## Development Tools

### Code Quality
- **Linting**: ESLint (frontend), Ruff (backend)
- **Formatting**: Prettier (frontend), Black (backend)
- **Type Checking**: TypeScript, mypy for Python
- **Pre-commit Hooks**: husky + lint-staged

### Testing Framework
- **Frontend Testing**: Vitest + React Testing Library
- **Backend Testing**: pytest + pytest-asyncio
- **E2E Testing**: Playwright (configured in PM system)
- **API Testing**: FastAPI TestClient, httpx

### Build & Deployment
- **CI/CD**: GitHub Actions
- **Package Manager**: pnpm (frontend), pip + pip-tools (backend)
- **Environment Management**: Docker containers + environment variables
- **Deployment Strategy**: Blue-green deployment

## Integration Technologies

### Payment Processing
- **China Market**: 
  - 支付宝 (Alipay) SDK integration
  - 微信支付 (WeChat Pay) API
  - 银联支付 for traditional cards
- **International**: 
  - Stripe for global payments
  - PayPal for broader coverage

### Third-Party Services
- **Email**: SendGrid or AWS SES
- **SMS**: 阿里云短信服务 or Twilio
- **Search**: MySQL full-text + Redis for advanced search
- **Analytics**: Custom analytics + Google Analytics 4
- **CDN**: CloudFlare for performance and security

## Security Technologies

### Application Security
- **Authentication**: JWT with refresh token rotation
- **Authorization**: Role-Based Access Control (RBAC)
- **API Security**: Rate limiting, CORS, input validation
- **Data Protection**: Encryption at rest and in transit

### Infrastructure Security
- **Network Security**: VPC, security groups, firewalls
- **Certificate Management**: Automated SSL/TLS certificate management
- **Secret Management**: Environment variables, Docker secrets
- **Compliance**: GDPR, 个人信息保护法 compliance measures

## Development Environment

### Local Development Stack
```bash
# Frontend Development
cd frontend && pnpm dev          # Next.js dev server on :3000
cd mobile && expo start         # Expo dev server on :8081

# Backend Development  
cd backend && uvicorn app.main:app --reload  # FastAPI on :8000

# Database Services
docker-compose up mysql redis    # Local database services
```

### Required Software
- **Node.js**: 18.17+ LTS
- **Python**: 3.11+
- **Docker**: 24.0+ with Docker Compose
- **Git**: 2.40+
- **Redis**: 7.0+ (via Docker)
- **MySQL**: 8.0+ (via Docker)

## Performance Targets

### Application Performance
- **First Contentful Paint**: < 1.5s
- **Time to Interactive**: < 2.5s
- **API Response Time**: < 500ms (95th percentile)
- **Database Query Time**: < 100ms average

### Scalability Targets
- **Concurrent Users**: 10,000+
- **Content Storage**: Unlimited (object storage)
- **Database Size**: Multi-TB support
- **API Throughput**: 1000+ RPS per server

## Compliance & Standards

### Code Standards
- **Frontend**: ESLint Airbnb config, Prettier
- **Backend**: PEP 8 with Black formatting, Ruff linting
- **API Design**: RESTful principles, OpenAPI 3.0
- **Database**: Normalized design with proper indexing

### Regulatory Compliance
- **Data Privacy**: GDPR, 个人信息保护法
- **Payment Compliance**: PCI DSS for payment processing
- **Content Regulations**: 网络安全法 compliance
- **Accessibility**: WCAG 2.1 AA compliance