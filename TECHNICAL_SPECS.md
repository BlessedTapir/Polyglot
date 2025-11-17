# Polyglot Technical Specifications

**Version:** 1.0
**Date:** November 17, 2025
**Status:** Draft

---

## Table of Contents
1. [Overview](#overview)
2. [System Architecture](#system-architecture)
3. [Technology Stack](#technology-stack)
4. [Data Models](#data-models)
5. [API Specifications](#api-specifications)
6. [Frontend Architecture](#frontend-architecture)
7. [Security](#security)
8. [Performance Requirements](#performance-requirements)
9. [Infrastructure](#infrastructure)
10. [Development & Deployment](#development--deployment)

---

## Overview

### Purpose
This document defines the technical architecture and implementation details for the Polyglot platform MVP.

### System Goals
- **Scalability:** Support 10,000+ concurrent users
- **Performance:** <2s page load, <200ms API response
- **Reliability:** 99.5% uptime
- **Security:** Secure user data and prevent common vulnerabilities
- **Maintainability:** Clean, documented, testable code

### Technical Principles
- RESTful API design
- Mobile-first responsive design
- Stateless authentication
- Database normalization
- Test-driven development
- Continuous integration/deployment

---

## System Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                          CLIENTS                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Browser    │  │    Mobile    │  │   Tablet     │      │
│  │   (React)    │  │  (Responsive)│  │ (Responsive) │      │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘      │
│         │                  │                  │               │
│         └──────────────────┴──────────────────┘               │
│                            │                                  │
│                     HTTPS / REST API                          │
│                            │                                  │
└────────────────────────────┼──────────────────────────────────┘
                             │
┌────────────────────────────┼──────────────────────────────────┐
│                            │                                  │
│                    ┌───────▼────────┐                         │
│                    │   API Gateway  │                         │
│                    │   (Nginx/ALB)  │                         │
│                    └───────┬────────┘                         │
│                            │                                  │
│         ┌──────────────────┴─────────────────┐               │
│         │                                     │               │
│   ┌─────▼──────┐                    ┌────────▼────────┐      │
│   │   Auth     │                    │   Application   │      │
│   │  Service   │◄───────────────────┤     Server      │      │
│   │  (JWT)     │                    │   (Node.js/     │      │
│   └─────┬──────┘                    │    Python)      │      │
│         │                            └────────┬────────┘      │
│         │                                     │               │
│         │         ┌───────────────────────────┤               │
│         │         │                           │               │
│   ┌─────▼─────────▼───┐            ┌─────────▼──────────┐    │
│   │    PostgreSQL     │            │     Redis Cache    │    │
│   │    (Primary DB)   │            │   (Sessions/Rate)  │    │
│   └───────────────────┘            └────────────────────┘    │
│                                                               │
│                          BACKEND LAYER                        │
└───────────────────────────────────────────────────────────────┘
                             │
┌────────────────────────────┼──────────────────────────────────┐
│                            │                                  │
│   ┌────────────────────────▼──────────────────────┐          │
│   │            Background Jobs Queue              │          │
│   │              (Celery / Bull)                  │          │
│   │  ┌─────────────┐  ┌──────────────────────┐   │          │
│   │  │Pilot Group  │  │ Difficulty Scoring   │   │          │
│   │  │Processor    │  │    Calculator        │   │          │
│   │  └─────────────┘  └──────────────────────┘   │          │
│   └───────────────────────────────────────────────┘          │
│                                                               │
│                    WORKER/JOBS LAYER                          │
└───────────────────────────────────────────────────────────────┘
```

### Component Overview

#### Client Layer
- **Web Application:** Single Page Application (SPA) using React
- **Responsive Design:** Works on desktop, tablet, and mobile browsers
- **State Management:** Redux or Context API
- **Routing:** React Router

#### API Layer
- **API Gateway:** Nginx or cloud load balancer
- **Rate Limiting:** Prevent abuse
- **SSL/TLS:** All traffic encrypted
- **CORS:** Configured for web clients

#### Application Layer
- **Application Server:** Node.js (Express) or Python (Django/FastAPI)
- **Business Logic:** Modular service architecture
- **Authentication:** JWT-based stateless auth
- **Validation:** Request/response validation

#### Data Layer
- **Primary Database:** PostgreSQL (relational data)
- **Cache:** Redis (sessions, rate limiting, hot data)
- **File Storage:** S3 or equivalent (future: user avatars, images)

#### Worker Layer
- **Job Queue:** Celery (Python) or Bull (Node.js)
- **Background Tasks:** Pilot group processing, score calculation, notifications

---

## Technology Stack

### ✅ Selected Stack (Python/FastAPI)

**Decision Date:** November 17, 2025
**Rationale:** FastAPI provides rapid development, excellent async support, automatic API documentation, and strong type hints with Pydantic.

#### Frontend
- **Framework:** React 18+
- **Language:** TypeScript
- **State Management:** Redux Toolkit or Zustand
- **Styling:** Tailwind CSS + shadcn/ui components
- **Build Tool:** Vite
- **Testing:** Jest + React Testing Library
- **E2E Testing:** Playwright

#### Backend
- **Language:** Python 3.11+
- **Framework:** FastAPI
- **ORM:** SQLAlchemy 2.0+ (async)
- **Validation:** Pydantic v2
- **Testing:** Pytest + pytest-asyncio
- **Task Queue:** Celery + Redis
- **API Documentation:** FastAPI automatic docs (Swagger UI + ReDoc)
- **ASGI Server:** Uvicorn
- **Database Migrations:** Alembic

#### Database
- **Primary:** PostgreSQL 15+
- **Cache:** Redis 7+
- **Migration Tool:** Alembic

#### Infrastructure
- **Hosting:** Vercel (frontend) + Railway/Render (backend)
- **Or:** AWS (EC2, RDS, ElastiCache)
- **Or:** Google Cloud Platform
- **CI/CD:** GitHub Actions
- **Monitoring:** Sentry + Uptime Robot
- **Analytics:** Plausible or Google Analytics

---

### Why FastAPI?

✅ **Advantages for Polyglot:**
1. **Speed:** High performance with async/await support
2. **Development Velocity:** Automatic API docs, less boilerplate
3. **Type Safety:** Pydantic models provide runtime validation
4. **Modern Python:** Uses Python 3.11+ features
5. **Easy Testing:** Pytest integration is seamless
6. **Learning Curve:** Pythonic and intuitive
7. **Community:** Growing ecosystem and active development

### Alternative Considered

**Node.js + TypeScript** was considered but not selected for this project. Both options are viable for production use.

---

## Data Models

### Entity Relationship Diagram

```
┌──────────────┐         ┌──────────────┐         ┌──────────────┐
│    Users     │         │    Teams     │         │  Questions   │
├──────────────┤         ├──────────────┤         ├──────────────┤
│ id (PK)      │    ┌────┤ id (PK)      │         │ id (PK)      │
│ username     │    │    │ name         │         │ title        │
│ email        │    │    │ description  │         │ content      │
│ password_hash│    │    │ total_points │         │ language     │
│ team_id (FK) │────┘    │ created_at   │    ┌────┤ creator_id   │
│ role         │         └──────────────┘    │    │ difficulty   │
│ points       │                             │    │ status       │
│ created_at   │         ┌──────────────┐    │    │ created_at   │
│ updated_at   │         │   Answers    │    │    └──────────────┘
└──────────────┘         ├──────────────┤    │            │
       │                 │ id (PK)      │    │            │
       │                 │ question_id  │────┘            │
       │                 │ content      │                 │
       │                 │ is_correct   │                 │
       │                 │ created_at   │                 │
       │                 └──────────────┘                 │
       │                                                  │
       │                 ┌──────────────┐                 │
       │                 │  Responses   │                 │
       │                 ├──────────────┤                 │
       │                 │ id (PK)      │                 │
       └─────────────────┤ user_id (FK) │                 │
                         │ question_id  │─────────────────┘
                         │ answer_id    │
                         │ is_correct   │
                         │ time_taken   │
                         │ created_at   │
                         └──────────────┘
                                │
                         ┌──────────────┐
                         │ PilotGroups  │
                         ├──────────────┤
                         │ id (PK)      │
                         │ question_id  │
                         │ target_size  │
                         │ current_size │
                         │ status       │
                         │ avg_score    │
                         │ difficulty   │
                         │ created_at   │
                         └──────────────┘
```

### Database Schema

#### Users Table
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    team_id INTEGER REFERENCES teams(id),
    role VARCHAR(20) DEFAULT 'user', -- 'user', 'moderator', 'admin'
    points INTEGER DEFAULT 0,
    avatar_url VARCHAR(500),
    bio TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    last_login TIMESTAMP,
    is_active BOOLEAN DEFAULT true,
    is_verified BOOLEAN DEFAULT false
);

CREATE INDEX idx_users_username ON users(username);
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_team_id ON users(team_id);
```

#### Teams Table
```sql
CREATE TABLE teams (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) UNIQUE NOT NULL,
    description TEXT,
    total_points INTEGER DEFAULT 0,
    member_count INTEGER DEFAULT 0,
    avatar_url VARCHAR(500),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_teams_name ON teams(name);
CREATE INDEX idx_teams_points ON teams(total_points DESC);
```

#### Questions Table
```sql
CREATE TABLE questions (
    id SERIAL PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    content TEXT NOT NULL,
    language VARCHAR(50) NOT NULL, -- 'python', 'javascript', 'java', etc.
    creator_id INTEGER REFERENCES users(id) ON DELETE SET NULL,
    difficulty DECIMAL(3,2), -- 0.00 to 1.00, calculated from pilot group
    status VARCHAR(20) DEFAULT 'draft', -- 'draft', 'pilot', 'active', 'archived'
    tags TEXT[], -- Array of tags
    explanation TEXT, -- Optional explanation for educational purposes
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    views INTEGER DEFAULT 0,
    attempts INTEGER DEFAULT 0,
    correct_attempts INTEGER DEFAULT 0
);

CREATE INDEX idx_questions_language ON questions(language);
CREATE INDEX idx_questions_status ON questions(status);
CREATE INDEX idx_questions_creator ON questions(creator_id);
CREATE INDEX idx_questions_difficulty ON questions(difficulty);
```

#### Answers Table
```sql
CREATE TABLE answers (
    id SERIAL PRIMARY KEY,
    question_id INTEGER REFERENCES questions(id) ON DELETE CASCADE,
    content TEXT NOT NULL,
    is_correct BOOLEAN NOT NULL,
    display_order INTEGER NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_answers_question_id ON answers(question_id);
```

#### Responses Table
```sql
CREATE TABLE responses (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
    question_id INTEGER REFERENCES questions(id) ON DELETE CASCADE,
    answer_id INTEGER REFERENCES answers(id) ON DELETE CASCADE,
    is_correct BOOLEAN NOT NULL,
    time_taken INTEGER, -- seconds
    points_awarded INTEGER DEFAULT 0,
    is_pilot BOOLEAN DEFAULT false, -- true if part of pilot group
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_responses_user_id ON responses(user_id);
CREATE INDEX idx_responses_question_id ON responses(question_id);
CREATE INDEX idx_responses_created_at ON responses(created_at);
CREATE UNIQUE INDEX idx_responses_user_question ON responses(user_id, question_id);
```

#### PilotGroups Table
```sql
CREATE TABLE pilot_groups (
    id SERIAL PRIMARY KEY,
    question_id INTEGER UNIQUE REFERENCES questions(id) ON DELETE CASCADE,
    target_size INTEGER DEFAULT 100, -- Target number of responses
    current_size INTEGER DEFAULT 0, -- Current number of responses
    status VARCHAR(20) DEFAULT 'active', -- 'active', 'completed', 'failed'
    avg_score DECIMAL(5,2), -- Average score 0-100
    difficulty DECIMAL(3,2), -- Calculated difficulty 0.00-1.00
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    completed_at TIMESTAMP
);

CREATE INDEX idx_pilot_groups_status ON pilot_groups(status);
CREATE INDEX idx_pilot_groups_question_id ON pilot_groups(question_id);
```

#### PointTransactions Table (Audit Trail)
```sql
CREATE TABLE point_transactions (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
    team_id INTEGER REFERENCES teams(id) ON DELETE CASCADE,
    amount INTEGER NOT NULL, -- Can be negative
    type VARCHAR(50) NOT NULL, -- 'quiz_correct', 'quiz_incorrect', 'question_created', etc.
    reference_id INTEGER, -- ID of related entity (question_id, response_id, etc.)
    description TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_point_transactions_user_id ON point_transactions(user_id);
CREATE INDEX idx_point_transactions_team_id ON point_transactions(team_id);
CREATE INDEX idx_point_transactions_created_at ON point_transactions(created_at);
```

---

## API Specifications

### API Design Principles
- RESTful architecture
- JSON request/response bodies
- Consistent error handling
- Versioned endpoints (v1, v2, etc.)
- Pagination for list endpoints
- Rate limiting

### Authentication
- JWT (JSON Web Tokens)
- Access token (short-lived, 15 minutes)
- Refresh token (long-lived, 7 days)
- Bearer token in Authorization header

### Base URL
```
Development: http://localhost:3000/api/v1
Production: https://api.polyglot.app/v1
```

### Standard Response Format

#### Success Response
```json
{
  "success": true,
  "data": { /* response data */ },
  "meta": {
    "page": 1,
    "limit": 20,
    "total": 100
  }
}
```

#### Error Response
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": [
      {
        "field": "email",
        "message": "Email is required"
      }
    ]
  }
}
```

### API Endpoints

#### Authentication Endpoints

**POST /auth/register**
```json
Request:
{
  "username": "string (3-50 chars)",
  "email": "string (valid email)",
  "password": "string (min 8 chars)"
}

Response: 201 Created
{
  "success": true,
  "data": {
    "user": {
      "id": 1,
      "username": "john_doe",
      "email": "john@example.com",
      "team_id": null,
      "role": "user"
    },
    "accessToken": "eyJhbGc...",
    "refreshToken": "eyJhbGc..."
  }
}
```

**POST /auth/login**
```json
Request:
{
  "email": "string",
  "password": "string"
}

Response: 200 OK
{
  "success": true,
  "data": {
    "user": { /* user object */ },
    "accessToken": "eyJhbGc...",
    "refreshToken": "eyJhbGc..."
  }
}
```

**POST /auth/refresh**
```json
Request:
{
  "refreshToken": "string"
}

Response: 200 OK
{
  "success": true,
  "data": {
    "accessToken": "eyJhbGc...",
    "refreshToken": "eyJhbGc..."
  }
}
```

**POST /auth/logout**
```json
Request: (requires auth token)
{}

Response: 200 OK
{
  "success": true,
  "message": "Logged out successfully"
}
```

---

#### User Endpoints

**GET /users/me**
```json
Response: 200 OK
{
  "success": true,
  "data": {
    "id": 1,
    "username": "john_doe",
    "email": "john@example.com",
    "team": {
      "id": 1,
      "name": "Team Alpha"
    },
    "points": 1250,
    "role": "user",
    "created_at": "2025-01-01T00:00:00Z"
  }
}
```

**PATCH /users/me**
```json
Request:
{
  "username": "string (optional)",
  "bio": "string (optional)",
  "avatar_url": "string (optional)"
}

Response: 200 OK
{
  "success": true,
  "data": { /* updated user object */ }
}
```

**GET /users/:id**
```json
Response: 200 OK
{
  "success": true,
  "data": {
    "id": 1,
    "username": "john_doe",
    "team": { /* team object */ },
    "points": 1250,
    "bio": "Love coding!",
    "stats": {
      "questions_created": 5,
      "quizzes_taken": 50,
      "correct_rate": 0.75
    }
  }
}
```

---

#### Question Endpoints

**POST /questions**
```json
Request:
{
  "title": "string (required, max 255)",
  "content": "string (required)",
  "language": "string (required)",
  "tags": ["array", "of", "strings"],
  "explanation": "string (optional)",
  "answers": [
    {
      "content": "string (required)",
      "is_correct": boolean (required)
    }
  ] // Minimum 2, maximum 6 answers, exactly one must be correct
}

Response: 201 Created
{
  "success": true,
  "data": {
    "id": 1,
    "title": "What is a closure?",
    "status": "pilot",
    "pilot_group": {
      "id": 1,
      "target_size": 100,
      "current_size": 0
    }
  }
}
```

**GET /questions**
```json
Query Parameters:
- language: string (filter by language)
- status: string (filter by status)
- difficulty_min: number (0-1)
- difficulty_max: number (0-1)
- page: number (default: 1)
- limit: number (default: 20, max: 100)
- sort: string (created_at, difficulty, attempts)

Response: 200 OK
{
  "success": true,
  "data": [
    {
      "id": 1,
      "title": "What is a closure?",
      "language": "javascript",
      "difficulty": 0.65,
      "attempts": 150,
      "correct_rate": 0.45,
      "creator": {
        "id": 1,
        "username": "john_doe"
      }
    }
  ],
  "meta": {
    "page": 1,
    "limit": 20,
    "total": 100
  }
}
```

**GET /questions/:id**
```json
Response: 200 OK
{
  "success": true,
  "data": {
    "id": 1,
    "title": "What is a closure?",
    "content": "A closure is...",
    "language": "javascript",
    "difficulty": 0.65,
    "status": "active",
    "tags": ["functions", "scope"],
    "answers": [
      {
        "id": 1,
        "content": "A function that...",
        "display_order": 1
      },
      // is_correct not included for active questions
    ],
    "creator": {
      "id": 1,
      "username": "john_doe"
    },
    "created_at": "2025-01-01T00:00:00Z"
  }
}
```

**POST /questions/:id/respond**
```json
Request:
{
  "answer_id": number (required)
}

Response: 200 OK
{
  "success": true,
  "data": {
    "is_correct": true,
    "points_awarded": 10,
    "correct_answer_id": 2,
    "explanation": "A closure is...",
    "user_stats": {
      "total_points": 1260,
      "correct_rate": 0.76
    }
  }
}
```

---

#### Team Endpoints

**GET /teams**
```json
Query Parameters:
- sort: string (points, member_count, name)
- page: number
- limit: number

Response: 200 OK
{
  "success": true,
  "data": [
    {
      "id": 1,
      "name": "Team Alpha",
      "total_points": 15000,
      "member_count": 25,
      "rank": 1
    }
  ],
  "meta": { /* pagination */ }
}
```

**GET /teams/:id**
```json
Response: 200 OK
{
  "success": true,
  "data": {
    "id": 1,
    "name": "Team Alpha",
    "description": "The best team!",
    "total_points": 15000,
    "member_count": 25,
    "rank": 1,
    "top_members": [
      {
        "id": 1,
        "username": "john_doe",
        "points": 1250
      }
    ]
  }
}
```

**POST /teams/:id/join**
```json
Request: {} (empty body)

Response: 200 OK
{
  "success": true,
  "data": {
    "team": { /* team object */ },
    "message": "Successfully joined Team Alpha"
  }
}
```

---

#### Leaderboard Endpoints

**GET /leaderboard/users**
```json
Query Parameters:
- period: string (all, week, month) - default: all
- page: number
- limit: number

Response: 200 OK
{
  "success": true,
  "data": [
    {
      "rank": 1,
      "user": {
        "id": 1,
        "username": "john_doe",
        "team": { /* team object */ }
      },
      "points": 5000,
      "badges": ["top_contributor", "quiz_master"]
    }
  ],
  "meta": { /* pagination */ }
}
```

**GET /leaderboard/teams**
```json
Response: Similar to /teams but with rank information
```

---

### Rate Limiting

| Endpoint Pattern | Rate Limit | Window |
|-----------------|------------|---------|
| /auth/login | 5 requests | 15 minutes |
| /auth/register | 3 requests | 1 hour |
| /questions (POST) | 10 requests | 1 hour |
| /questions/:id/respond | 100 requests | 1 hour |
| Other authenticated | 100 requests | 15 minutes |
| Other public | 50 requests | 15 minutes |

---

## Frontend Architecture

### Component Structure

```
src/
├── components/
│   ├── common/           # Reusable UI components
│   │   ├── Button/
│   │   ├── Input/
│   │   ├── Modal/
│   │   └── Card/
│   ├── layout/           # Layout components
│   │   ├── Header/
│   │   ├── Footer/
│   │   ├── Sidebar/
│   │   └── MainLayout/
│   ├── features/         # Feature-specific components
│   │   ├── auth/
│   │   │   ├── LoginForm/
│   │   │   └── RegisterForm/
│   │   ├── questions/
│   │   │   ├── QuestionList/
│   │   │   ├── QuestionCard/
│   │   │   ├── QuestionForm/
│   │   │   └── QuizInterface/
│   │   ├── teams/
│   │   │   ├── TeamList/
│   │   │   └── TeamCard/
│   │   └── leaderboard/
│   │       └── LeaderboardTable/
├── pages/                # Page components
│   ├── HomePage/
│   ├── LoginPage/
│   ├── QuizPage/
│   ├── CreateQuestionPage/
│   ├── LeaderboardPage/
│   └── ProfilePage/
├── hooks/                # Custom React hooks
│   ├── useAuth.ts
│   ├── useQuestions.ts
│   └── useTeams.ts
├── services/             # API services
│   ├── api.ts           # Axios instance
│   ├── authService.ts
│   ├── questionService.ts
│   └── teamService.ts
├── store/                # State management
│   ├── authSlice.ts
│   ├── questionSlice.ts
│   └── store.ts
├── types/                # TypeScript types
│   ├── user.ts
│   ├── question.ts
│   └── team.ts
├── utils/                # Utility functions
│   ├── validators.ts
│   └── formatters.ts
└── App.tsx
```

### Key Pages

1. **Home Page** - Landing page with overview and featured questions
2. **Login/Register Pages** - Authentication
3. **Quiz Page** - Take a quiz on a specific question
4. **Browse Questions Page** - List and filter questions
5. **Create Question Page** - Form to create new questions
6. **Profile Page** - User profile and stats
7. **Team Page** - Team information and members
8. **Leaderboard Page** - User and team rankings

### State Management

**Global State (Redux/Zustand):**
- User authentication status
- Current user data
- Team information
- Notifications/alerts

**Local State:**
- Form inputs
- UI state (modals, dropdowns)
- Pagination state

**Server State (React Query):**
- Questions list
- Leaderboard data
- Team data
- User profiles

---

## Security

### Authentication & Authorization

#### Password Security
- Minimum 8 characters
- Hashed with bcrypt (cost factor 12)
- Never logged or exposed in responses

#### JWT Security
- Short-lived access tokens (15 minutes)
- Refresh tokens stored in httpOnly cookies
- Refresh token rotation
- Token blacklist for logout

#### Authorization Levels
- **Guest:** Browse public questions, view leaderboard
- **User:** All guest + take quizzes, create questions, join teams
- **Moderator:** All user + edit/delete questions, moderate content
- **Admin:** All moderator + manage users, teams, system settings

### Input Validation

#### Backend Validation
- Validate all inputs against schema
- Sanitize HTML/SQL inputs
- Check file upload types and sizes
- Rate limit all endpoints

#### Frontend Validation
- Client-side validation for UX
- Never trust client validation alone
- Validate on blur and submit

### Security Best Practices

#### OWASP Top 10 Mitigation

1. **Injection:**
   - Use parameterized queries (ORM)
   - Validate and sanitize all inputs
   - Use prepared statements

2. **Broken Authentication:**
   - Strong password policy
   - Multi-factor authentication (future)
   - Session timeout
   - Account lockout after failed attempts

3. **Sensitive Data Exposure:**
   - HTTPS everywhere
   - Encrypt sensitive data at rest
   - Don't expose unnecessary data in APIs

4. **XML External Entities (XXE):**
   - Disable XML external entity processing
   - Use JSON instead of XML

5. **Broken Access Control:**
   - Check authorization on every request
   - Deny by default
   - Log access control failures

6. **Security Misconfiguration:**
   - Remove default accounts
   - Disable directory listing
   - Keep dependencies updated

7. **Cross-Site Scripting (XSS):**
   - Escape user-generated content
   - Use Content Security Policy
   - Sanitize HTML inputs

8. **Insecure Deserialization:**
   - Validate serialized objects
   - Use JSON instead of pickle/serialize

9. **Using Components with Known Vulnerabilities:**
   - Regular dependency audits
   - Automated security scanning
   - Keep all libraries updated

10. **Insufficient Logging & Monitoring:**
    - Log all authentication events
    - Monitor for suspicious activity
    - Alert on security events

#### Headers
```
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Content-Security-Policy: default-src 'self'
```

---

## Performance Requirements

### Response Time Targets

| Metric | Target | Maximum |
|--------|--------|---------|
| Page Load Time (FCP) | <1.5s | <2s |
| Time to Interactive (TTI) | <3s | <5s |
| API Response (p95) | <150ms | <200ms |
| Database Query (p95) | <50ms | <100ms |

### Optimization Strategies

#### Frontend
- Code splitting and lazy loading
- Image optimization (WebP, lazy load)
- Bundle size optimization (<200KB initial)
- CDN for static assets
- Service worker caching (future)

#### Backend
- Database query optimization
- Index critical columns
- Connection pooling
- Redis caching for hot data
- Query result caching

#### Database
- Proper indexing strategy
- Query optimization
- Regular VACUUM (PostgreSQL)
- Connection pooling
- Read replicas for scaling (future)

### Scalability Targets

| Metric | MVP | 6 Months | 1 Year |
|--------|-----|----------|---------|
| Concurrent Users | 100 | 1,000 | 10,000 |
| Daily Active Users | 50 | 500 | 5,000 |
| Questions in DB | 1,000 | 10,000 | 50,000 |
| API Requests/day | 10K | 100K | 1M |

---

## Infrastructure

### Development Environment

```yaml
docker-compose.yml:
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

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

  backend:
    build: ./backend
    ports:
      - "3000:3000"
    environment:
      DATABASE_URL: postgresql://polyglot:dev_password@postgres:5432/polyglot_dev
      REDIS_URL: redis://redis:6379
    depends_on:
      - postgres
      - redis

  frontend:
    build: ./frontend
    ports:
      - "5173:5173"
    volumes:
      - ./frontend:/app
```

### Production Infrastructure (AWS Example)

```
┌─────────────────────────────────────────────────────┐
│                   Route 53 (DNS)                    │
└────────────────────┬────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────┐
│              CloudFront (CDN)                       │
│         (Static Assets + Frontend)                  │
└────────────────────┬────────────────────────────────┘
                     │
       ┌─────────────┴──────────────┐
       │                            │
┌──────▼──────┐            ┌────────▼─────────┐
│   S3 Bucket │            │  Load Balancer   │
│  (Frontend) │            │   (ALB/NLB)      │
└─────────────┘            └────────┬─────────┘
                                    │
                    ┌───────────────┴────────────────┐
                    │                                │
           ┌────────▼─────────┐          ┌──────────▼────────┐
           │  EC2/ECS/Fargate │          │  EC2/ECS/Fargate  │
           │  (Backend App)   │          │  (Backend App)    │
           └────────┬─────────┘          └──────────┬────────┘
                    │                               │
           ┌────────▼────────────────────────────────▼────────┐
           │              RDS (PostgreSQL)                    │
           │          (Multi-AZ for HA)                       │
           └──────────────────────────────────────────────────┘
                    │
           ┌────────▼────────┐
           │  ElastiCache    │
           │    (Redis)      │
           └─────────────────┘
```

### Recommended Hosting Solutions

#### Option 1: Vercel + Railway (Easiest)
- **Frontend:** Vercel (automatic deployments)
- **Backend:** Railway (container deployment)
- **Database:** Railway PostgreSQL
- **Cache:** Railway Redis
- **Cost:** ~$20-50/month
- **Best For:** MVP, small teams

#### Option 2: AWS (Most Control)
- **Frontend:** S3 + CloudFront
- **Backend:** ECS Fargate or EC2
- **Database:** RDS PostgreSQL
- **Cache:** ElastiCache Redis
- **Cost:** ~$100-300/month
- **Best For:** Scaling, enterprise

#### Option 3: Google Cloud Platform
- **Frontend:** Firebase Hosting or Cloud Storage
- **Backend:** Cloud Run or App Engine
- **Database:** Cloud SQL (PostgreSQL)
- **Cache:** Memorystore (Redis)
- **Cost:** ~$80-250/month
- **Best For:** Google ecosystem integration

---

## Development & Deployment

### Development Workflow

1. **Feature Branch**
   ```bash
   git checkout -b feature/quiz-interface
   ```

2. **Local Development**
   - Run tests: `npm test`
   - Run linter: `npm run lint`
   - Local server: `npm run dev`

3. **Pull Request**
   - Create PR on GitHub
   - Automated checks run (CI)
   - Code review required
   - Merge to main

4. **Deployment**
   - Automatic deployment to staging
   - Manual promotion to production

### CI/CD Pipeline (GitHub Actions)

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '20'
      - run: npm ci
      - run: npm run lint
      - run: npm test
      - run: npm run build

  deploy-staging:
    needs: test
    if: github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to staging
        run: |
          # Deploy commands

  deploy-production:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to production
        run: |
          # Deploy commands
```

### Environment Variables

#### Backend (.env)
```bash
# Database
DATABASE_URL=postgresql://user:pass@host:5432/dbname

# Redis
REDIS_URL=redis://host:6379

# JWT
JWT_SECRET=your-secret-key
JWT_EXPIRES_IN=15m
REFRESH_TOKEN_SECRET=your-refresh-secret
REFRESH_TOKEN_EXPIRES_IN=7d

# API
API_PORT=3000
NODE_ENV=development

# CORS
CORS_ORIGIN=http://localhost:5173

# Rate Limiting
RATE_LIMIT_WINDOW=15
RATE_LIMIT_MAX_REQUESTS=100
```

#### Frontend (.env)
```bash
VITE_API_URL=http://localhost:3000/api/v1
VITE_APP_NAME=Polyglot
VITE_ENABLE_ANALYTICS=false
```

---

## Monitoring & Observability

### Metrics to Track

#### Application Metrics
- Request rate (requests/second)
- Error rate (%)
- Response time (p50, p95, p99)
- Database query time
- Cache hit rate

#### Business Metrics
- Daily/Monthly Active Users (DAU/MAU)
- Questions created per day
- Quizzes taken per day
- User retention rate
- Team participation rate

#### Infrastructure Metrics
- CPU usage
- Memory usage
- Disk I/O
- Network I/O
- Database connections

### Logging Strategy

#### Log Levels
- **ERROR:** System errors, exceptions
- **WARN:** Deprecated features, slow queries
- **INFO:** User actions, API calls
- **DEBUG:** Detailed diagnostic info

#### What to Log
- ✅ Authentication events
- ✅ API requests (exclude sensitive data)
- ✅ Database errors
- ✅ Background job execution
- ❌ Passwords or tokens
- ❌ Personal information (unless necessary)

### Recommended Tools
- **Application Monitoring:** Sentry, New Relic, DataDog
- **Uptime Monitoring:** Uptime Robot, Pingdom
- **Log Aggregation:** Logtail, Papertrail
- **Analytics:** Plausible, Mixpanel

---

## Testing Strategy

### Test Pyramid

```
        ┌─────────────┐
        │   E2E Tests │  (10%)
        └─────────────┘
       ┌───────────────┐
       │Integration Tests│ (30%)
       └───────────────┘
      ┌─────────────────┐
      │   Unit Tests    │  (60%)
      └─────────────────┘
```

### Unit Tests
- Test individual functions and components
- Mock external dependencies
- Target: >80% coverage

### Integration Tests
- Test API endpoints
- Test database operations
- Test service interactions

### End-to-End Tests
- Test critical user flows:
  - User registration and login
  - Creating a question
  - Taking a quiz
  - Joining a team

### Testing Tools
- **Unit:** Jest (Backend), Jest + RTL (Frontend)
- **Integration:** Supertest (Backend)
- **E2E:** Playwright or Cypress
- **Load Testing:** k6 or Artillery

---

## Appendix

### Glossary
- **MVP:** Minimum Viable Product
- **JWT:** JSON Web Token
- **ORM:** Object-Relational Mapping
- **TTI:** Time to Interactive
- **FCP:** First Contentful Paint
- **DAU:** Daily Active Users
- **MAU:** Monthly Active Users

### References
- [REST API Design Best Practices](https://restfulapi.net/)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [React Best Practices](https://react.dev/)

### Related Documents
- [Project Plan](PROJECT_PLAN.md)
- [MVP Requirements](MVP_REQUIREMENTS.md)
- [Project Status](PROJECT_STATUS.md)

---

**Document Version:** 1.0
**Last Updated:** November 17, 2025
**Author:** Claude

*This document should be reviewed and updated as technical decisions are finalized.*
