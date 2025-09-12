---
created: 2025-09-12T05:26:33Z
last_updated: 2025-09-12T05:26:33Z
version: 1.0
author: Claude Code PM System
---

# Project Style Guide

## Code Standards & Conventions

ShareStack项目遵循现代软件工程最佳实践，确保代码质量、可维护性和团队协作效率。

## Language-Specific Standards

### TypeScript/JavaScript (Frontend)

#### Naming Conventions
```typescript
// Variables & Functions: camelCase
const userName = "zhangsan";
const getUserProfile = async (userId: string) => { ... };

// Constants: SCREAMING_SNAKE_CASE
const API_BASE_URL = "https://api.sharestack.com";
const MAX_UPLOAD_SIZE = 10 * 1024 * 1024;

// Interfaces & Types: PascalCase
interface UserProfile {
  id: string;
  email: string;
  displayName: string;
}

type ContentStatus = 'draft' | 'published' | 'archived';

// Components: PascalCase
const ContentEditor = ({ content }: ContentEditorProps) => { ... };
const UserProfileCard = () => { ... };

// Files: kebab-case
user-profile.tsx
content-editor.tsx
api-client.ts
```

#### Code Organization
```typescript
// Import order: External libraries → Internal modules → Relative imports
import React, { useState, useEffect } from 'react';
import { NextPage } from 'next';
import { Button } from '@/components/ui/button';
import { useAuth } from '@/lib/auth';
import { ContentService } from './content-service';

// Function signatures with proper typing
interface CreateContentProps {
  title: string;
  content: string;
  tags: string[];
  publishAt?: Date;
}

const createContent = async ({ 
  title, 
  content, 
  tags, 
  publishAt 
}: CreateContentProps): Promise<Content> => {
  // Implementation
};
```

#### Error Handling Standards
```typescript
// Use Result pattern for error handling
type Result<T, E = Error> = 
  | { success: true; data: T }
  | { success: false; error: E };

// Async functions should handle errors explicitly
const fetchUserContent = async (userId: string): Promise<Result<Content[]>> => {
  try {
    const response = await api.get(`/users/${userId}/content`);
    return { success: true, data: response.data };
  } catch (error) {
    return { success: false, error: error as Error };
  }
};
```

### Python (Backend)

#### Naming Conventions  
```python
# Variables & Functions: snake_case
user_name = "zhangsan"
async def get_user_profile(user_id: str) -> UserProfile:
    ...

# Constants: SCREAMING_SNAKE_CASE  
API_BASE_URL = "https://api.sharestack.com"
MAX_UPLOAD_SIZE = 10 * 1024 * 1024

# Classes: PascalCase
class UserService:
    def __init__(self, user_repo: UserRepository):
        self.user_repo = user_repo

# Files: snake_case
user_profile.py
content_service.py
payment_gateway.py
```

#### Type Annotations
```python
from typing import Optional, List, Dict, Any, Union
from pydantic import BaseModel

# All function signatures must include type hints
async def create_user(
    user_data: UserCreate,
    db: AsyncSession = Depends(get_db)
) -> User:
    """Create a new user with proper validation."""
    user = User(**user_data.dict())
    db.add(user)
    await db.commit()
    await db.refresh(user)
    return user

# Use Pydantic models for data validation
class ContentCreate(BaseModel):
    title: str = Field(..., min_length=1, max_length=200)
    content: str = Field(..., min_length=10)
    tags: List[str] = Field(default=[], max_items=10)
    is_premium: bool = False
    
    @validator('title')
    def title_must_not_be_empty(cls, v):
        if not v.strip():
            raise ValueError('Title cannot be empty')
        return v.strip()
```

#### Error Handling Standards
```python
# Custom exception hierarchy
class ShareStackException(Exception):
    """Base exception for ShareStack application."""
    pass

class ValidationError(ShareStackException):
    """Raised when data validation fails."""
    pass

class AuthenticationError(ShareStackException):
    """Raised when authentication fails.""" 
    pass

# Structured error responses
@app.exception_handler(ValidationError)
async def validation_exception_handler(request: Request, exc: ValidationError):
    return JSONResponse(
        status_code=422,
        content={
            "success": False,
            "error": {
                "type": "validation_error",
                "message": str(exc),
                "timestamp": datetime.utcnow().isoformat()
            }
        }
    )
```

## File & Directory Organization

### Frontend Structure
```
frontend/
├── app/                    # Next.js App Router
│   ├── (auth)/            # Route groups  
│   ├── dashboard/         # Dashboard pages
│   ├── content/           # Content pages
│   └── globals.css        # Global styles
├── components/            # Reusable components
│   ├── ui/               # Basic UI components (shadcn/ui)
│   ├── forms/            # Form components
│   ├── layouts/          # Layout components
│   └── content/          # Content-specific components
├── lib/                  # Utilities & configurations
│   ├── auth.ts           # Authentication utilities
│   ├── api.ts            # API client
│   ├── utils.ts          # General utilities
│   └── validation.ts     # Validation schemas
├── types/                # TypeScript type definitions
│   ├── user.ts          # User-related types
│   ├── content.ts       # Content-related types
│   └── api.ts           # API response types
└── hooks/               # Custom React hooks
    ├── use-auth.ts      # Authentication hooks
    ├── use-content.ts   # Content management hooks
    └── use-api.ts       # API hooks
```

### Backend Structure
```
backend/
├── app/
│   ├── api/             # API routes
│   │   ├── auth/        # Authentication endpoints
│   │   ├── users/       # User management endpoints
│   │   ├── content/     # Content management endpoints
│   │   └── payments/    # Payment endpoints
│   ├── core/           # Core functionality
│   │   ├── config.py   # Application configuration
│   │   ├── database.py # Database connection
│   │   ├── auth.py     # Authentication logic
│   │   └── exceptions.py # Custom exceptions
│   ├── models/         # Database models
│   │   ├── user.py     # User model
│   │   ├── content.py  # Content model
│   │   └── payment.py  # Payment model
│   ├── schemas/        # Pydantic schemas
│   │   ├── user.py     # User schemas
│   │   ├── content.py  # Content schemas
│   │   └── payment.py  # Payment schemas
│   ├── services/       # Business logic
│   │   ├── user_service.py     # User operations
│   │   ├── content_service.py  # Content operations
│   │   └── payment_service.py  # Payment operations
│   └── utils/          # Utility functions
├── tests/              # Test files
│   ├── test_auth.py    # Authentication tests
│   ├── test_content.py # Content tests
│   └── conftest.py     # Test configuration
├── alembic/           # Database migrations
└── requirements.txt   # Python dependencies
```

## Coding Standards

### Code Formatting

#### Frontend (TypeScript/JavaScript)
- **Formatter**: Prettier with these settings:
```json
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 80,
  "tabWidth": 2,
  "useTabs": false
}
```

#### Backend (Python)
- **Formatter**: Black with line length 88
- **Import Sorting**: isort
- **Linting**: Ruff with strict settings

### Comment Standards

#### Function Documentation
```typescript
/**
 * Creates a new content piece with validation and user permission checks.
 * 
 * @param contentData - The content creation data
 * @param userId - ID of the user creating the content  
 * @returns Promise resolving to the created content
 * @throws ValidationError when content data is invalid
 * @throws AuthenticationError when user lacks permissions
 */
const createContent = async (
  contentData: ContentCreate, 
  userId: string
): Promise<Content> => {
  // Implementation
};
```

```python
async def create_content(
    content_data: ContentCreate,
    user_id: str,
    db: AsyncSession
) -> Content:
    """
    Create a new content piece with validation and permission checks.
    
    Args:
        content_data: The content creation data
        user_id: ID of the user creating the content
        db: Database session
        
    Returns:
        The created content object
        
    Raises:
        ValidationError: When content data is invalid
        AuthenticationError: When user lacks permissions
    """
    # Implementation
```

#### Code Comments
```typescript
// GOOD: Explain WHY, not WHAT
// Use exponential backoff to handle rate limiting gracefully
const retryWithBackoff = async (fn: () => Promise<any>, maxRetries = 3) => {
  // Implementation
};

// BAD: Explain obvious code
// Increment the counter by 1
counter += 1;
```

### Database Standards

#### Model Definitions
```python
# SQLAlchemy models with proper constraints and relationships
class User(Base):
    __tablename__ = "users"
    
    id = Column(UUID(as_uuid=True), primary_key=True, default=uuid.uuid4)
    email = Column(String(255), unique=True, index=True, nullable=False)
    password_hash = Column(String(255), nullable=False)
    display_name = Column(String(100), nullable=False)
    is_active = Column(Boolean, default=True, nullable=False)
    is_verified = Column(Boolean, default=False, nullable=False)
    created_at = Column(DateTime, default=datetime.utcnow, nullable=False)
    updated_at = Column(
        DateTime, 
        default=datetime.utcnow, 
        onupdate=datetime.utcnow, 
        nullable=False
    )
    
    # Relationships with proper lazy loading
    contents = relationship(
        "Content", 
        back_populates="author",
        lazy="select"
    )
    subscriptions = relationship(
        "Subscription",
        back_populates="user",
        lazy="select"
    )
```

#### Migration Standards
```python
# Alembic migration with proper up/down operations
def upgrade():
    """Add user verification status."""
    op.add_column('users', sa.Column('is_verified', sa.Boolean(), nullable=False, server_default='false'))
    op.create_index('ix_users_is_verified', 'users', ['is_verified'])

def downgrade():
    """Remove user verification status."""
    op.drop_index('ix_users_is_verified', table_name='users')
    op.drop_column('users', 'is_verified')
```

## Testing Standards

### Test Organization
```typescript
// Frontend: Component testing with React Testing Library
describe('ContentEditor', () => {
  beforeEach(() => {
    // Setup
  });

  it('should create content when valid data is provided', async () => {
    // Arrange
    const mockContent = { title: 'Test', content: 'Test content' };
    
    // Act
    render(<ContentEditor />);
    // ... user interactions
    
    // Assert
    expect(screen.getByText('Content created')).toBeInTheDocument();
  });
});
```

```python
# Backend: pytest with async support
@pytest.mark.asyncio
class TestContentService:
    async def test_create_content_success(self, db_session, test_user):
        """Test successful content creation."""
        # Arrange
        content_data = ContentCreate(
            title="Test Content",
            content="This is test content"
        )
        
        # Act
        result = await ContentService(db_session).create_content(
            content_data, test_user.id
        )
        
        # Assert
        assert result.title == "Test Content"
        assert result.author_id == test_user.id
```

## Documentation Standards

### API Documentation
```python
@router.post("/content/", response_model=ContentResponse)
async def create_content(
    content: ContentCreate,
    current_user: User = Depends(get_current_user),
    db: AsyncSession = Depends(get_db)
):
    """
    Create a new content piece.
    
    - **title**: Content title (1-200 characters)
    - **content**: Main content body (minimum 10 characters)
    - **tags**: Optional tags for categorization
    - **is_premium**: Whether content requires subscription
    
    Returns the created content with generated ID and metadata.
    """
    return await ContentService(db).create_content(content, current_user.id)
```

### README Standards
Each module should include:
- Purpose and scope
- Installation/setup instructions
- Usage examples
- API documentation links
- Testing instructions
- Contributing guidelines

## Quality Assurance

### Pre-commit Checks
```bash
# Frontend
npm run lint          # ESLint
npm run type-check     # TypeScript
npm run test          # Jest/Vitest
npm run format        # Prettier

# Backend
ruff check .          # Python linting
black --check .       # Code formatting
mypy .                # Type checking
pytest               # Unit tests
```

### Code Review Checklist
- [ ] Code follows naming conventions
- [ ] All functions have proper type annotations
- [ ] Error handling is comprehensive
- [ ] Tests cover new functionality
- [ ] Documentation is updated
- [ ] No hardcoded values or secrets
- [ ] Performance considerations addressed
- [ ] Security implications reviewed

这些标准确保ShareStack项目在团队协作中保持代码一致性、可维护性和高质量。所有开发者都应遵循这些规范，并通过代码审查确保标准得到执行。