# Project Structure

## Organization Philosophy

Mastodon follows **Rails conventions with domain-driven organization**:
- Backend: Standard Rails MVC + Service layer pattern
- Frontend: Feature-first React/Redux architecture
- Clear separation between API layer, business logic, and presentation

## Directory Patterns

### Backend (Ruby on Rails)

#### Controllers (`app/controllers/`)
**Purpose**: HTTP request handling, routing to services
**Pattern**: API versioned under `api/v1/`, `api/v2/`, admin routes under `admin/`
**Example**: `api/v1/statuses_controller.rb`, `admin/accounts_controller.rb`

#### Models (`app/models/`)
**Purpose**: Domain entities, ActiveRecord persistence
**Pattern**: One file per model, concerns in `concerns/`, form objects in `form/`
**Example**: `account.rb`, `status.rb`, `concerns/account_associations.rb`

#### Services (`app/services/`)
**Purpose**: Business logic, complex operations
**Pattern**: One service per operation, inherits from `BaseService`
**Example**: `post_status_service.rb`, `follow_service.rb`, `activitypub/process_*_service.rb`

#### Libraries (`app/lib/`)
**Purpose**: Framework-agnostic logic, utilities
**Pattern**: Domain-organized subdirectories
**Example**: `activitypub/`, `translation_service/`, `webhooks/`

#### Serializers (`app/serializers/`)
**Purpose**: JSON API response formatting
**Pattern**: ActiveModelSerializers, mirroring model structure
**Example**: `rest/account_serializer.rb`, `rest/status_serializer.rb`

#### Workers (`app/workers/`)
**Purpose**: Background job processing (Sidekiq)
**Pattern**: One worker per job type, scheduled jobs marked
**Example**: `distribution_worker.rb`, `feed_insert_worker.rb`

### Frontend (React/TypeScript)

#### Entrypoints (`app/javascript/entrypoints/`)
**Purpose**: Vite entry points for different pages
**Pattern**: One file per page/bundle
**Example**: `application.ts` (main SPA), `admin.tsx`, `public.tsx`

#### Mastodon App (`app/javascript/mastodon/`)
**Purpose**: Main React application code
**Pattern**: Feature-first organization with shared components

##### Features (`mastodon/features/`)
**Purpose**: Page-level components and feature modules
**Pattern**: One directory per feature with components, containers
**Example**: `compose/`, `notifications/`, `status/`, `home_timeline/`

##### Actions (`mastodon/actions/`)
**Purpose**: Redux action creators and thunks
**Pattern**: One file per domain, typed actions in `*_typed.ts`
**Example**: `accounts.js`, `statuses_typed.ts`, `compose.js`

##### Reducers (`mastodon/reducers/`)
**Purpose**: Redux state reducers
**Pattern**: Domain-based slices, combined in index
**Example**: `accounts.js`, `statuses.js`, `compose.js`

##### Components (`mastodon/components/`)
**Purpose**: Reusable UI components
**Pattern**: Presentational components, no business logic
**Example**: `button.tsx`, `status.jsx`, `avatar.tsx`

##### API (`mastodon/api/`)
**Purpose**: API client layer
**Pattern**: Typed API functions, centralized error handling
**Example**: `accounts.ts`, `statuses.ts`, base client in `api.ts`

## Naming Conventions

### Ruby/Rails
- **Files**: `snake_case.rb`
- **Classes**: `PascalCase` (e.g., `AccountSearchService`)
- **Methods**: `snake_case` (e.g., `find_account`)
- **Constants**: `SCREAMING_SNAKE_CASE`

### JavaScript/TypeScript
- **Files**: `snake_case.ts` (legacy) → `kebab-case.tsx` (newer code)
- **Components**: `PascalCase` (e.g., `StatusContent.tsx`)
- **Functions**: `camelCase` (e.g., `fetchAccount`)
- **Constants**: `SCREAMING_SNAKE_CASE` or `camelCase` for config

## Import Organization

### Rails
```ruby
# Standard library first
require 'json'

# Gems second
require 'sidekiq'

# Application code third
require_relative '../lib/custom_helper'
```

### TypeScript/React
```typescript
// External dependencies first
import React from 'react';
import { connect } from 'react-redux';

// Internal modules with path aliases
import { fetchAccount } from '@/mastodon/actions/accounts';
import { Avatar } from 'mastodon/components/avatar';

// Relative imports last
import './styles.scss';
```

**Path Aliases**:
- `@/*`: Maps to `app/javascript/*`
- `mastodon/*`: Maps to `app/javascript/mastodon/*`
- `images/*`, `styles/*`: Asset directories

## Code Organization Principles

### Separation of Concerns
- **Controllers**: Thin, delegate to services
- **Models**: Data + associations, keep queries simple
- **Services**: Business logic, complex operations
- **Workers**: Async processing, idempotent operations

### Dependency Flow
- Controllers → Services → Models
- Services can call other services
- Models should not depend on controllers/services
- Workers → Services (for reusability)

### Federation Isolation
ActivityPub code isolated in:
- `app/lib/activitypub/`: Protocol implementation
- `app/services/activitypub/`: Federation-specific services
- `app/workers/activitypub/`: Federation jobs

### API Versioning
- Namespace controllers: `Api::V1::`, `Api::V2::`
- Maintain backward compatibility within version
- Serializers versioned alongside controllers

### Frontend State Management
- **Global State**: Redux (accounts, statuses, timelines)
- **UI State**: React hooks (modals, dropdowns, form inputs)
- **Server Cache**: Redux as cache, invalidate on mutations

---
_Document patterns, not file trees. New files following patterns shouldn't require updates_
