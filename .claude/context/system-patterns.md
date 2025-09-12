---
created: 2025-09-12T05:26:33Z
last_updated: 2025-09-12T05:26:33Z
version: 1.0
author: Claude Code PM System
---

# System Patterns

## Architectural Patterns

ShareStack follows modern full-stack architectural patterns designed for scalability, maintainability, and developer productivity.

### Overall Architecture Pattern: **Microservices-Ready Monolith**

The system starts as a well-structured monolith with clear service boundaries, designed for easy extraction into microservices as scale requirements grow.

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Frontend      │    │   Mobile App    │    │   Admin Panel   │
│   (Next.js)     │    │  (React Native) │    │   (Next.js)     │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         └─────────────┬─────────────────────────────────┘
                       │
                ┌─────────────────┐
                │   API Gateway   │
                │   (FastAPI)     │
                └─────────────────┘
                       │
      ┌────────────────┼────────────────┐
      │                │                │
┌─────────────┐ ┌─────────────┐ ┌─────────────┐
│   User      │ │  Content    │ │  Payment    │
│  Service    │ │  Service    │ │  Service    │
└─────────────┘ └─────────────┘ └─────────────┘
      │                │                │
      └────────────────┼────────────────┘
                       │
                ┌─────────────────┐
                │   Database      │
                │  (MySQL + Redis)│
                └─────────────────┘
```

## Backend Service Patterns

### 1. **Domain-Driven Design (DDD)**
Services organized around business domains:
- **User Domain**: Authentication, profiles, subscriptions
- **Content Domain**: Articles, media, publishing workflow
- **Payment Domain**: Subscriptions, transactions, payouts
- **Analytics Domain**: Metrics, reporting, insights

### 2. **Repository Pattern**
Data access abstraction with SQLAlchemy:
```python
class UserRepository:
    async def create_user(self, user_data: UserCreate) -> User:
        # Database operation abstracted
    
    async def get_user_by_email(self, email: str) -> Optional[User]:
        # Query logic encapsulated
```

### 3. **Service Layer Pattern**
Business logic separation from API controllers:
```python
class UserService:
    def __init__(self, user_repo: UserRepository, email_service: EmailService):
        self.user_repo = user_repo
        self.email_service = email_service
    
    async def register_user(self, user_data: UserCreate) -> User:
        # Business logic here
```

### 4. **Dependency Injection**
FastAPI's built-in DI system for loose coupling:
```python
@app.post("/users/")
async def create_user(
    user_data: UserCreate,
    user_service: UserService = Depends(get_user_service)
):
    return await user_service.register_user(user_data)
```

## Frontend Architecture Patterns

### 1. **Component Composition Pattern**
React components designed for maximum reusability:
```tsx
// Compound component pattern
<ContentEditor>
  <ContentEditor.Toolbar />
  <ContentEditor.TextArea />
  <ContentEditor.StatusBar />
</ContentEditor>
```

### 2. **Container/Presentational Pattern**
Clear separation between logic and presentation:
- **Container Components**: Data fetching, state management
- **Presentational Components**: UI rendering, user interactions

### 3. **Custom Hooks Pattern**
Reusable logic extraction:
```tsx
// Custom hook for content operations
function useContentEditor(contentId: string) {
  const { data, mutate } = useMutation(saveContent);
  const { data: content } = useQuery(['content', contentId], fetchContent);
  
  return { content, saveContent: mutate, isLoading };
}
```

### 4. **State Management Pattern**
Zustand for predictable state management:
```tsx
const useAuthStore = create((set, get) => ({
  user: null,
  login: async (credentials) => {
    const user = await authApi.login(credentials);
    set({ user });
  },
  logout: () => set({ user: null })
}));
```

## Data Flow Patterns

### 1. **Unidirectional Data Flow**
```
User Action → API Call → Backend Service → Database → Response → UI Update
```

### 2. **Event-Driven Updates**
For real-time features:
```
Content Published → Event Emitted → Subscribers Notified → UI Updates
```

### 3. **Optimistic Updates**
UI updates immediately, rollback on failure:
```tsx
const mutation = useMutation(updateContent, {
  onMutate: async (newContent) => {
    // Optimistically update UI
    queryClient.setQueryData(['content', id], newContent);
  },
  onError: (err, newContent, context) => {
    // Rollback on error
    queryClient.setQueryData(['content', id], context.previousContent);
  }
});
```

## Database Design Patterns

### 1. **Active Record Pattern** (via SQLAlchemy ORM)
Models encapsulate both data and behavior:
```python
class User(Base):
    __tablename__ = "users"
    
    id = Column(Integer, primary_key=True)
    email = Column(String, unique=True, index=True)
    
    def can_access_content(self, content_id: int) -> bool:
        # Business logic in model
```

### 2. **Soft Delete Pattern**
Preserve data integrity:
```python
class BaseModel(Base):
    __abstract__ = True
    deleted_at = Column(DateTime, nullable=True)
    
    def soft_delete(self):
        self.deleted_at = datetime.utcnow()
```

### 3. **Audit Pattern**
Track all changes:
```python
class AuditMixin:
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)
    created_by = Column(Integer, ForeignKey("users.id"))
```

## Security Patterns

### 1. **JWT Token Pattern**
Stateless authentication with secure token handling:
```python
# Access token (short-lived) + Refresh token (long-lived)
def create_access_token(user_id: int) -> str:
    expire = datetime.utcnow() + timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    return jwt.encode({"user_id": user_id, "exp": expire}, SECRET_KEY)
```

### 2. **Role-Based Access Control (RBAC)**
```python
class Permission(Enum):
    READ_CONTENT = "read:content"
    WRITE_CONTENT = "write:content"
    MANAGE_USERS = "manage:users"

@require_permission(Permission.WRITE_CONTENT)
async def create_content(content: ContentCreate, user: User = Depends(get_current_user)):
    # Protected endpoint
```

### 3. **Input Validation Pattern**
Pydantic models for comprehensive validation:
```python
class ContentCreate(BaseModel):
    title: str = Field(..., min_length=1, max_length=200)
    content: str = Field(..., min_length=10)
    tags: List[str] = Field(default=[], max_items=10)
    
    @validator('title')
    def title_must_not_be_empty(cls, v):
        if not v.strip():
            raise ValueError('Title cannot be empty')
        return v
```

## Error Handling Patterns

### 1. **Centralized Error Handling**
FastAPI exception handlers:
```python
@app.exception_handler(ValidationError)
async def validation_exception_handler(request: Request, exc: ValidationError):
    return JSONResponse(
        status_code=422,
        content={"error": "Validation failed", "details": exc.errors()}
    )
```

### 2. **Error Boundary Pattern** (Frontend)
React error boundaries for graceful error handling:
```tsx
class ContentErrorBoundary extends React.Component {
  componentDidCatch(error, errorInfo) {
    // Log error to monitoring service
    errorReporting.captureException(error);
  }
  
  render() {
    if (this.state.hasError) {
      return <ErrorFallback />;
    }
    return this.props.children;
  }
}
```

## Caching Patterns

### 1. **Multi-Layer Caching**
```
Browser Cache → CDN Cache → Redis Cache → Database
```

### 2. **Cache-Aside Pattern**
```python
async def get_user_profile(user_id: int) -> UserProfile:
    # Try cache first
    cached = await redis.get(f"user:{user_id}")
    if cached:
        return UserProfile.parse_raw(cached)
    
    # Fallback to database
    profile = await db.fetch_user_profile(user_id)
    await redis.set(f"user:{user_id}", profile.json(), ex=3600)
    return profile
```

## Testing Patterns

### 1. **Test Pyramid Pattern**
- **Unit Tests**: 70% - Fast, isolated tests
- **Integration Tests**: 20% - API and database tests  
- **E2E Tests**: 10% - Complete user workflow tests

### 2. **Arrange-Act-Assert Pattern**
```python
async def test_create_user():
    # Arrange
    user_data = UserCreate(email="test@example.com", password="secure123")
    
    # Act
    result = await user_service.create_user(user_data)
    
    # Assert
    assert result.email == "test@example.com"
    assert result.id is not None
```

### 3. **Factory Pattern for Test Data**
```python
class UserFactory:
    @staticmethod
    def create(**kwargs) -> User:
        defaults = {
            "email": "user@example.com",
            "password": "password123",
            "is_active": True
        }
        defaults.update(kwargs)
        return User(**defaults)
```

## Monitoring & Observability Patterns

### 1. **Structured Logging**
```python
import structlog

logger = structlog.get_logger()

async def create_content(content_data: ContentCreate, user: User):
    logger.info("Content creation started", 
                user_id=user.id, 
                content_type=content_data.type)
    # ... operation
    logger.info("Content created successfully", 
                user_id=user.id, 
                content_id=result.id)
```

### 2. **Health Check Pattern**
```python
@app.get("/health")
async def health_check():
    checks = {
        "database": await check_database_connection(),
        "redis": await check_redis_connection(),
        "storage": await check_storage_access()
    }
    
    status = "healthy" if all(checks.values()) else "unhealthy"
    return {"status": status, "checks": checks}
```

These patterns provide a robust foundation for building scalable, maintainable, and secure applications while maintaining clear separation of concerns and following modern software engineering best practices.