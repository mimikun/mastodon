# Mastodon Personal Fork - CLAUDE.md

## Project Overview

This is a personal fork of [Mastodon](https://github.com/mastodon/mastodon) for running a private instance server.

### Fork Information

- **Upstream Repository**: https://github.com/mastodon/mastodon
- **Fork Base**: an upstream **release tag** (currently `v4.7.0`), not a branch
- **Purpose**: Personal server deployment with custom modifications

### Key Points

- This fork contains custom modifications specific to personal server requirements
- Changes should be carefully documented for easier upstream merging if needed
- **Never merge upstream into `mimikun`.** Keep the branch as "one upstream
  release tag + the custom commits, and nothing else". Following a new release
  is then a single `git rebase --onto`.
- **The procedure lives in `FORK-MAINTENANCE.md`. Read that before updating.**
  It covers the remotes, the rebase, the meaning of each `--onto` argument, and
  the verification steps.

Tracking an upstream **branch** was tried and abandoned on 2026-06-21: version
bumps and Crowdin translations produced conflicts on every sync. This document
described that abandoned setup until 2026-08-09.

## Technology Stack

### Backend

- **Ruby on Rails**: REST API and web interface
- **Ruby Version**: 3.2+ (see `.ruby-version`)
- **PostgreSQL**: 14+ (main database)
- **Redis**: 7.0+ (caching and background jobs)
- **Sidekiq**: Background job processing

### Frontend

- **React**: 18.2.0 (UI library)
- **Redux**: State management (@reduxjs/toolkit)
- **TypeScript**: 5.9.0 (type safety)
- **Vite**: 7.1.1 (build tool and dev server)
- **Node.js**: 20+ (see `.nvmrc`)

### Package Managers

- **Yarn**: 4.10.3 (JavaScript packages) - **DO NOT use npm or pnpm**
- **Bundler**: Ruby gem management

### Code Quality Tools

#### Ruby

- **RuboCop**: Linting and style checking
- **RSpec**: Testing framework
- **Haml-Lint**: Template linting

#### JavaScript/TypeScript

- **ESLint**: Linting (`yarn lint:js`)
- **Stylelint**: CSS/SCSS linting (`yarn lint:css`)
- **Prettier**: Code formatting (`yarn format`)
- **TypeScript**: Type checking (`yarn typecheck`)
- **Vitest**: Testing framework (`yarn test:js`)

#### Other

- **Storybook**: UI component development and testing
- **Chromatic**: Visual regression testing

## Development Environment Setup

### Prerequisites

1. **Ruby**: 3.2+ (use rbenv, rvm, or asdf)
2. **Node.js**: 20+ (use nvm, fnm, or asdf)
3. **PostgreSQL**: 14+
4. **Redis**: 7.0+
5. **ImageMagick/libvips**: Image processing

### Quick Start (Docker)

```bash
# Start development environment
docker compose -f .devcontainer/compose.yaml up -d

# Setup database and dependencies
docker compose -f .devcontainer/compose.yaml exec app bin/setup

# Start development server
docker compose -f .devcontainer/compose.yaml exec app bin/dev
```

### Native Development (Linux/macOS)

```bash
# Install Ruby version from .ruby-version
rbenv install $(cat .ruby-version)
rbenv local $(cat .ruby-version)

# Install Node.js version from .nvmrc
nvm install
nvm use

# Install dependencies
bundle install
yarn install

# Setup database
RAILS_ENV=development bin/setup

# Start development server
bin/dev
```

### Default Admin Credentials

After setup, you can log in with:
- **Username**: `admin@localhost` or `admin@mastodon.local`
- **Password**: `mastodonadmin`

## Development Workflow

### Running Servers

```bash
# Start all services (Rails, Webpack, Sidekiq, Streaming)
bin/dev

# Or manually:
# Terminal 1: Rails server
bundle exec rails server

# Terminal 2: Vite dev server
yarn dev

# Terminal 3: Sidekiq (background jobs)
bundle exec sidekiq

# Terminal 4: Streaming API
yarn start
```

### Code Quality Checks

```bash
# Ruby linting and formatting
bundle exec rubocop --autocorrect-all

# JavaScript/TypeScript linting
yarn lint:js
yarn lint:css

# Format code with Prettier
yarn format

# Type checking
yarn typecheck

# Run all checks
yarn lint && yarn typecheck
```

### Testing

```bash
# Ruby tests (RSpec)
bundle exec rspec

# Specific test file
bundle exec rspec spec/models/account_spec.rb

# JavaScript tests (Vitest)
yarn test:js

# Storybook tests
yarn test:storybook

# Run Storybook locally
yarn storybook
```

### Database Operations

```bash
# Create database
bundle exec rails db:create

# Run migrations
bundle exec rails db:migrate

# Rollback migration
bundle exec rails db:rollback

# Reset database (WARNING: destroys all data)
bundle exec rails db:reset

# Populate with sample data for development
bundle exec rails dev:populate_sample_data
```

## Coding Standards

### Ruby Guidelines

- Follow [RuboCop](https://rubocop.org/) guidelines (see `.rubocop.yml`)
- Use Ruby 3.2+ syntax features
- Prefer keyword arguments for methods with multiple parameters
- Use strong parameters for all controller actions
- Write RSpec tests for all business logic

### JavaScript/TypeScript Guidelines

- Use TypeScript for all new code
- Enable `strict` mode in TypeScript
- Follow ESLint rules (see `eslint.config.mjs`)
- Use functional components and hooks in React
- Prefer immutable data structures
- Write tests for UI components using Vitest

### Style Conventions

- **Indentation**: 2 spaces (Ruby, JavaScript, SCSS)
- **Line Length**:
  - Ruby: Flexible (see RuboCop config)
  - JavaScript/TypeScript: Managed by Prettier
- **Quotes**:
  - Ruby: Single quotes preferred
  - JavaScript: Prettier default (single)

### Commit Message Format

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

Types:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style changes (formatting, missing semicolons, etc.)
- `refactor`: Code refactoring
- `perf`: Performance improvements
- `test`: Adding or updating tests
- `chore`: Maintenance tasks

Examples:
```bash
git commit -m "feat(api): add custom emoji upload endpoint"
git commit -m "fix(ui): resolve timeline scrolling issue"
git commit -m "docs(readme): update installation instructions"
```

## Useful Commands

### Development

```bash
# Access Rails console
bundle exec rails console

# Access database console
bundle exec rails dbconsole

# Generate new migration
bundle exec rails generate migration AddFieldToModel

# View routes
bundle exec rails routes

# Clear cache
bundle exec rails cache:clear
```

### Asset Management

```bash
# Build assets for development
yarn build:development

# Build assets for production
yarn build:production

# Precompile assets (Rails)
bundle exec rails assets:precompile
```

### i18n (Internationalization)

```bash
# Extract new translatable strings
yarn i18n:extract

# Normalize locale files
bundle exec i18n-tasks normalize
```

## Upstream Synchronization

### Adding Upstream Remote

```bash
# Add upstream remote
git remote add upstream https://github.com/mastodon/mastodon.git

# Verify remotes
git remote -v
```

### Syncing with Upstream

```bash
# Fetch upstream changes
git fetch upstream

# View changes
git log HEAD..upstream/main --oneline

# Merge upstream changes (careful with custom modifications)
git merge upstream/main

# Or rebase (preferred for cleaner history)
git rebase upstream/main
```

### Handling Conflicts

1. Identify custom modifications before syncing
2. Document changes in commit messages
3. Resolve conflicts carefully, preserving custom features
4. Test thoroughly after merge/rebase
5. Consider creating feature branches for major customizations

### Tracking Custom Changes

```bash
# List commits not in upstream
git log upstream/main..HEAD

# Show diff from upstream
git diff upstream/main
```

## Customization Guidelines

### Where to Make Changes

- **App Logic**: `app/` directory
- **Configuration**: `config/` directory
- **Custom Styles**: `app/javascript/styles/` (SCSS)
- **Custom Components**: `app/javascript/mastodon/features/`
- **Database Schema**: Generate migrations in `db/migrate/`

### Best Practices for Customizations

1. **Branch Strategy**: Use feature branches for custom modifications
   ```bash
   git checkout -b custom/feature-name
   ```

2. **Documentation**: Document all custom changes
   - Add comments explaining why changes were made
   - Update this CLAUDE.md if needed

3. **Upstream Compatibility**:
   - Avoid modifying core files when possible
   - Use Rails concerns and decorators for extending functionality
   - Keep custom code in separate modules

4. **Testing**:
   - Write tests for all custom functionality
   - Ensure upstream tests still pass

5. **Configuration**:
   - Use environment variables for custom settings
   - Document required ENV vars in `.env.production.sample`

## Deployment

### Environment Variables

Create `.env.production` based on `.env.production.sample`:

```bash
cp .env.production.sample .env.production
```

Key variables to configure:
- `LOCAL_DOMAIN`: Your instance domain
- `SINGLE_USER_MODE`: Set to `true` for personal instance
- `SECRET_KEY_BASE`: Generate with `bundle exec rake secret`
- `OTP_SECRET`: Generate with `bundle exec rake secret`
- Database credentials (POSTGRES_*)
- Redis configuration (REDIS_*)
- SMTP settings for email
- S3/Object storage for media (optional but recommended)

### Building for Production

```bash
# Precompile assets
RAILS_ENV=production bundle exec rails assets:precompile

# Build JavaScript
yarn build:production

# Run database migrations
RAILS_ENV=production bundle exec rails db:migrate
```

### Docker Deployment

Use the provided `Dockerfile` and `docker-compose.yml`:

```bash
# Build image
docker compose build

# Start services
docker compose up -d

# Run migrations
docker compose exec web bundle exec rails db:migrate

# Create admin user
docker compose exec web tootctl accounts create \
  username \
  --email your@email.com \
  --confirmed \
  --role Owner
```

## Troubleshooting

### Common Issues

#### Assets not loading
```bash
# Clear cache and rebuild
bundle exec rails tmp:clear
yarn build:development
```

#### Database connection errors
```bash
# Check PostgreSQL is running
systemctl status postgresql  # Linux
brew services list            # macOS

# Verify database.yml configuration
cat config/database.yml
```

#### Redis connection errors
```bash
# Check Redis is running
systemctl status redis  # Linux
brew services list      # macOS
```

#### Sidekiq jobs not processing
```bash
# Restart Sidekiq
# Stop with Ctrl+C and restart:
bundle exec sidekiq
```

### Performance Tuning

- Enable Redis cache store in production
- Configure object storage for media files
- Use CDN for static assets
- Optimize PostgreSQL configuration
- Monitor Sidekiq queue sizes

## Additional Resources

- [Mastodon Documentation](https://docs.joinmastodon.org)
- [Mastodon API Documentation](https://docs.joinmastodon.org/api/)
- [ActivityPub Specification](https://www.w3.org/TR/activitypub/)
- [Ruby on Rails Guides](https://guides.rubyonrails.org)
- [React Documentation](https://react.dev)

## Project-Specific Notes

### Personal Instance Considerations

- **SINGLE_USER_MODE**: Recommended for personal instances
- **Moderation**: Simplified for single-user setup
- **Federation**: Be mindful of federation policies
- **Backups**: Regular backups of PostgreSQL database and media files
- **Updates**: Plan upgrade strategy (test in staging first)

### Custom Features Log

(Document your custom features here as you add them)

Example format:
```
## Custom Feature: [Feature Name]
- **Date Added**: YYYY-MM-DD
- **Branch**: custom/feature-name
- **Description**: Brief description of what this feature does
- **Files Modified**: List of key files changed
- **Upstream Compatibility**: Notes on potential conflicts
```

---

**Note**: This CLAUDE.md file should be updated as you add custom features or make significant changes to your development workflow.

