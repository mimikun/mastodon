# Technology Stack

## Architecture

Mastodon follows a **service-oriented Rails architecture** with a clear separation between:
- Backend API (Rails REST + Streaming)
- Frontend UI (React/Redux SPA)
- Background processing (Sidekiq)
- Federation layer (ActivityPub protocol)

## Core Technologies

- **Language**: Ruby 3.2+, JavaScript/TypeScript
- **Framework**: Ruby on Rails 8.0, React 18
- **Runtime**: Node.js 20+ (streaming server)
- **Database**: PostgreSQL 14+ (primary), Redis 7.0+ (cache/queue)
- **Package Managers**: Bundler (Ruby), Yarn 4 (JavaScript)

## Key Libraries

### Backend
- **OAuth**: Doorkeeper (OAuth2 provider)
- **Background Jobs**: Sidekiq + Sidekiq Scheduler
- **Federation**: Custom ActivityPub implementation
- **Authentication**: Devise + Two-Factor
- **File Uploads**: Paperclip with S3/Azure support
- **Search**: Chewy (Elasticsearch client)

### Frontend
- **State Management**: Redux Toolkit with Redux Immutable
- **Routing**: React Router 5
- **UI Components**: Custom component library
- **Build Tool**: Vite 7 with legacy browser support
- **Testing**: Vitest, Testing Library, Storybook
- **i18n**: react-intl with FormatJS

## Development Standards

### Type Safety
- TypeScript strict mode enabled (`strict: true`)
- Gradual migration from JS to TS (`.ts`, `.tsx` for new code)
- Path aliases: `@/*`, `mastodon/*`

### Code Quality
- **Ruby**: RuboCop, Brakeman (security)
- **JavaScript**: ESLint 9 (flat config), Prettier, Stylelint
- **Git Hooks**: Husky + lint-staged for pre-commit checks

### Testing
- **Ruby**: RSpec (Rails standard)
- **JavaScript**: Vitest (unit), Storybook (component), Playwright (e2e)
- **Coverage**: Expected for new features

## Development Environment

### Required Tools
- Ruby 3.2+, PostgreSQL 14+, Redis 7.0+, Node.js 20+
- Docker & docker-compose (optional but recommended)

### Common Commands
```bash
# Dev server (Vite)
yarn dev

# Build
yarn build:development  # Development build
yarn build:production   # Production build

# Quality checks
yarn lint               # ESLint + Stylelint
yarn typecheck          # TypeScript
yarn test               # Vitest

# Rails
bin/rails server        # API server
bin/rails console       # Rails console
bundle exec sidekiq     # Background workers
```

## Key Technical Decisions

### ActivityPub Federation
- Custom implementation in `app/lib/activitypub/`
- HTTP Signatures for authentication
- WebFinger for account discovery

### API Design
- RESTful API with versioning (`/api/v1`, `/api/v2`)
- OAuth2 with scoped permissions (Doorkeeper)
- Streaming API via Node.js WebSocket server

### Frontend Architecture
- **Pattern**: Feature-based organization (`app/javascript/mastodon/features/`)
- **State**: Redux for global state, React hooks for local state
- **Async**: Redux Thunks for API calls
- **Performance**: Code splitting with dynamic imports, Immutable.js for state

### Background Processing
- Sidekiq for async jobs (federation, media processing, cleanup)
- Scheduled jobs via Sidekiq Scheduler
- Job priorities and retry logic

### Storage Strategy
- **Database**: PostgreSQL (primary data, full-text search via pg_trgm)
- **Cache**: Redis (sessions, rate limiting, feed caching)
- **Files**: S3-compatible storage (media uploads, backups)
- **Search**: Elasticsearch (optional, via Chewy)

---
_Document standards and patterns, not every dependency_
