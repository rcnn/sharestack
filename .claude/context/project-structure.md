---
created: 2025-09-12T05:26:33Z
last_updated: 2025-09-15T02:31:10Z
version: 1.1
author: Claude Code PM System
---

# Project Structure

## Root Directory Organization

```
sharestack/
├── .claude/                          # Project management and AI context
│   ├── agents/                       # Specialized sub-agent definitions
│   ├── commands/                     # 66+ PM command implementations
│   ├── context/                      # Project context documentation
│   ├── epics/                        # Epic-level project tracking
│   ├── prds/                         # Product Requirements Documents
│   ├── rules/                        # Development and PM rules
│   └── scripts/                      # Automation and helper scripts
├── docs/                            # Project documentation
│   └── api/                         # Complete API documentation
├── frontend/                         # Next.js 14+ PC web application (planned)
├── backend/                          # FastAPI Python backend (planned)
├── mobile/                           # React Native + Expo mobile app (planned)
├── deploy/                          # Deployment configurations (planned)
└── tests/                           # End-to-end and integration tests (planned)
```

## Current Implementation Status

### ✅ Implemented Directories

#### `docs/api/` - Complete API Documentation
```
docs/api/
├── README.md                         # API overview and navigation
├── user-management.md                # User APIs (1,240 lines)
├── content-management.md             # Content APIs (2,043 lines)
├── subscription-payment.md           # Payment APIs (1,541 lines)
├── social-interaction.md             # Social APIs (1,096 lines)
├── data-analytics.md                 # Analytics APIs (1,566 lines)
├── openapi/                         # OpenAPI 3.0 specifications
│   ├── user-management.yaml         # User API specs
│   ├── data-analytics.yaml          # Analytics API specs
│   └── social-interaction.yaml      # Social API specs
└── postman/                         # Postman test collections
    └── user-management.json         # User API test collection
```

#### `.claude/` - Project Management Framework
```
.claude/
├── agents/                           # 4 specialized agents
│   ├── code-analyzer.md            # Bug hunting and code analysis
│   ├── file-analyzer.md            # Verbose file summarization
│   ├── parallel-worker.md          # Multi-stream coordination
│   └── test-runner.md              # Test execution with logging
├── commands/                         # Comprehensive command system
│   ├── code-rabbit.md              # CodeRabbit integration
│   ├── context/                    # Context management commands
│   │   ├── create.md               # Initialize project context
│   │   ├── prime.md                # Load context into session
│   │   └── update.md               # Update existing context
│   ├── pm/                         # Project management commands
│   │   ├── blocked.md              # Handle blocked tasks
│   │   ├── prd-*.md               # PRD management commands
│   │   └── [60+ other PM commands]
│   ├── prompt.md                   # Complex prompt handling
│   ├── re-init.md                  # CLAUDE.md regeneration
│   └── testing/                    # Testing framework commands
├── context/                          # Project documentation
│   ├── prototypes/                 # HTML UI prototypes
│   │   ├── pc/                     # 10 PC interface prototypes
│   │   └── mobile/                 # 10 mobile interface prototypes
│   └── README.md                   # Context system guide
├── prds/                            # Product requirements
│   └── prd.md                      # 25KB+ Chinese PRD document
├── rules/                           # Development guidelines
└── settings.local.json              # Permission and access control
```

### 🔄 Planned Directories (Not Yet Created)

#### `frontend/` - PC Web Application
**Technology**: Next.js 14+ with App Router
```
frontend/
├── app/                             # Next.js App Router
├── components/                      # Reusable UI components
├── lib/                            # Utilities and configurations
├── public/                         # Static assets
├── styles/                         # Global styles and Tailwind config
└── types/                          # TypeScript type definitions
```

#### `backend/` - API Server
**Technology**: Python 3.11+ with FastAPI
```
backend/
├── app/                            # Main application
│   ├── api/                        # API route handlers
│   ├── core/                       # Core functionality
│   ├── models/                     # Database models
│   └── services/                   # Business logic
├── alembic/                        # Database migrations
├── tests/                          # Backend tests
└── requirements.txt                # Python dependencies
```

#### `mobile/` - Mobile Application
**Technology**: React Native + Expo SDK 50+
```
mobile/
├── src/                            # Source code
│   ├── components/                 # Reusable components
│   ├── screens/                    # Screen components
│   ├── navigation/                 # Navigation setup
│   └── services/                   # API and service integrations
├── assets/                         # Images, fonts, etc.
└── expo/                          # Expo configuration
```

## File Organization Patterns

### Current Naming Conventions
- **Commands**: Kebab-case with domain prefix (`pm/issue-start.md`)
- **Agents**: Kebab-case descriptive names (`code-analyzer.md`)
- **Documentation**: Descriptive lowercase (`progress.md`, `tech-context.md`)
- **Configuration**: Standard names (`settings.local.json`, `CLAUDE.md`)

### Planned Code Organization
- **Components**: PascalCase (`UserProfile.tsx`, `ContentEditor.tsx`)
- **Utilities**: camelCase (`formatDate.ts`, `apiHelpers.ts`)
- **Pages/Routes**: kebab-case URLs matching file structure
- **Database**: snake_case for tables and columns (`user_profiles`, `content_metadata`)

## Key Structural Decisions

### Multi-Platform Architecture
- **Separation**: Distinct directories for web, mobile, and backend
- **Shared**: Common types and utilities via npm packages
- **Communication**: RESTful API contract between frontend/mobile and backend

### Project Management Integration
- **Context-Driven**: All development guided by `.claude/context/`
- **Command-Driven**: Development workflow through PM commands
- **Agent-Assisted**: Heavy lifting delegated to specialized sub-agents

### Documentation Strategy
- **Living Documentation**: Context files updated with project progress
- **Prototype-First**: HTML prototypes inform implementation
- **Spec-Driven**: PRD drives all development decisions

## Directory Creation Timeline

### ✅ Completed
1. ✅ `.claude/` PM framework initialized
2. ✅ `docs/api/` complete API documentation with 7,486 lines
3. ✅ OpenAPI specifications and Postman collections

### Immediate (Current Session)
1. Initialize backend FastAPI structure based on API docs
2. Create database models from API specifications
3. Set up development environment configuration

### Phase 1 (Next 2 weeks)
1. Complete backend API implementation following documentation
2. Frontend Next.js setup with API client generation
3. Database schema implementation and migrations
4. Docker development environment

### Phase 2 (MVP Development)
1. Implement feature-based directory organization
2. Add comprehensive test directory structure
3. Documentation and deployment configurations
4. CI/CD pipeline integration