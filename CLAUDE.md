# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

OWASP Juice Shop is an intentionally insecure web application designed for security training, awareness demos, CTFs, and as a testing ground for security tools. It encompasses vulnerabilities from the OWASP Top Ten and many other real-world security flaws.

**Important**: This is a deliberately vulnerable application. Code patterns here represent security anti-patterns that should NOT be replicated in production applications.

## Documents To Refer

ALWAYS refer below documents while working on this application

1. /docs/DEVSECOPS_TRAINING_WITH_JUICE_SHOP.txt
2. /docs/STAGE_EXPLAINED.txt

## Git Commit Guidelines

- DO NOT include the Claude Code signature and co-author attribution in commit messages for this repository.

## Documentation Guidelines

- ALWAYS write the new document in txt format , follow
  ✓ TXT format
  ✓ TL;DR at the top
  ✓ In /docs directory
  ✓ Skim-able layout
  ✓ LIMITED to 200 lines (max 500-600 based on use-case)
  ✓ MINIMAL decoration, use under the heading "=====" - No heavy borders, just clean headings
- Don't create README.md file until asked by the user

### Start Every Session With:

```

Before we start, read /docs/CURRENT_STATE.txt and summarize what you understand about our current progress.
```

"Before we start, read /docs/CURRENT_STATE.txt and /docs/SESSION_HANDOFF.txt to understand where we left off"

Required reading at session start:

1. /docs/CURRENT_STATE.txt - Get full context
2. /docs/SESSION_HANDOFF.txt - Understand last session
3. /docs/ChangeLog.txt

### End Every Session With:

```
Run /update-session-handoff to capture today's work
```

This command will:

- Interactively capture what was accomplished
- Update all context files (CURRENT_STATE, SESSION_HANDOFF, COMMANDS_LOG, DECISIONS, ISSUES)
- Prepare handoff for next session
- Maintain project memory

## Technology Stack

### Backend

- **Runtime**: Node.js 20-24 (TypeScript compiled to JavaScript)
- **Framework**: Express.js
- **Database**: SQLite3 (via Sequelize ORM)
- **API**: RESTful endpoints + Finale REST for auto-generated CRUD
- **Real-time**: Socket.io for WebSocket communication
- **Build**: TypeScript compiler (`tsc`)

### Frontend

- **Framework**: Angular 20 (standalone components)
- **UI**: Angular Material
- **State Management**: RxJS observables
- **Build**: Angular CLI with custom webpack configuration
- **Translation**: ngx-translate for i18n

## Development Commands

### Initial Setup

```bash
npm install                    # Installs all dependencies (backend + frontend)
                              # Automatically builds frontend and server via postinstall
```

### Running the Application

```bash
npm start                      # Start production build (requires prior build)
npm run serve                  # Start dev mode with ts-node (backend) + ng serve (frontend)
npm run serve:dev              # Start with ts-node-dev for hot reload
```

Application runs at `http://localhost:3000`

### Building

```bash
npm run build:frontend         # Build Angular frontend only
npm run build:server          # Compile TypeScript backend to build/
npm run package               # Create distributable package
```

### Testing

```bash
npm test                       # Run all tests (frontend unit + backend unit)
npm run test:server           # Backend unit tests (Mocha)
npm run test:api              # API integration tests (Frisby/Jest)
npm run cypress:open          # Open Cypress for e2e tests
npm run cypress:run           # Run Cypress e2e tests headless
```

### Linting and Code Quality

```bash
npm run lint                   # Lint backend (.ts) and frontend (TS + SCSS)
npm run lint:fix              # Auto-fix linting issues
npm run lint:config           # Validate YAML config against schema
```

**Code Style**: JavaScript Standard Style (via ESLint)

- All code must pass ESLint checks
- 2-space indentation
- No semicolons (except where required)

### Other Commands

```bash
npm run rsn                    # Run "Really Secure Notepad" utility
npm run sbom                   # Generate Software Bill of Materials (CycloneDX)
```

## Architecture and Code Structure

### Backend Structure

```
server.ts              # Main Express application entry point
app.ts                 # Application bootstrapper
routes/                # HTTP route handlers (64+ files)
  ├── login.ts        # Authentication endpoint
  ├── basket.ts       # Shopping basket operations
  ├── search.ts       # Product search with intentional vulnerabilities
  └── ...
models/                # Sequelize ORM models (24 models)
  ├── index.ts        # Database initialization
  ├── user.ts         # User model
  ├── product.ts      # Product catalog
  ├── challenge.ts    # CTF challenge tracking
  └── ...
lib/                   # Utility libraries
  ├── insecurity.ts   # Security utilities (some intentionally weak)
  ├── utils.ts        # General utilities
  ├── antiCheat.ts    # Challenge anti-cheat detection
  └── startup/        # Application startup routines
data/                  # Data creation and static content
config/                # YAML configuration files
  └── default.yml     # Default configuration (customizable)
```

### Frontend Structure

```
frontend/src/app/
  ├── Models/          # TypeScript interfaces/models
  ├── Services/        # Angular services for API communication
  ├── [feature]/       # Feature modules (basket, product, admin, etc.)
  │   ├── *.component.ts
  │   ├── *.component.html
  │   └── *.component.scss
  └── app.module.ts    # Root module
```

### Key Architectural Patterns

**Database Models**:

- Sequelize ORM with TypeScript
- All models defined in `models/` directory
- Relationships defined in `models/relations.ts`
- Database initialized via `datacreator.ts` with seed data

**API Endpoints**:

- Manual routes in `routes/` directory
- Auto-generated REST endpoints via Finale for CRUD operations
- Most endpoints in `server.ts` follow pattern: `app.use('/api/[resource]', [route-handler])`

**Challenge System**:

- Challenges tracked in `ChallengeModel`
- Challenge solving detected in various route handlers
- Anti-cheat mechanisms in `lib/antiCheat.ts`
- Challenge hints stored in `HintModel`

**Configuration**:

- YAML-based configuration in `config/` directory
- Schema validation via `config.schema.yml`
- Customizable via environment variables or config files
- Multiple config modes: default, CTF, tutorial, unsafe, etc.

**Security (Intentional Vulnerabilities)**:

- `lib/insecurity.ts` contains deliberately weak crypto/auth functions
- Routes contain various OWASP Top 10 vulnerabilities (XSS, SQLi, etc.)
- File upload vulnerabilities in designated endpoints
- Weak session management examples

## Testing Strategy

**Unit Tests**:

- Backend: Mocha + Chai in `test/server/`
- Frontend: Jasmine/Karma (Angular default)

**API Tests**:

- Frisby (Jest wrapper) in `test/api/`
- Tests for all REST endpoints
- Coverage tracked with nyc/Istanbul

**E2E Tests**:

- Cypress in root `cypress/` directory
- Tests for user flows and challenge solutions
- Required for all new/changed challenges

## Docker Support

```bash
docker pull bkimminich/juice-shop
docker run --rm -p 3000:3000 bkimminich/juice-shop
```

Multi-stage Dockerfile:

1. `installer` stage: Builds application
2. `distroless/nodejs` stage: Minimal production image

## Configuration and Customization

Configuration files in `config/`:

- `default.yml` - Base configuration
- `ctf.yml` - CTF mode overrides
- `tutorial.yml` - Tutorial mode settings

Key configuration sections:

- `server`: Port and base URL settings
- `application`: Branding, theme, feature flags
- `challenges`: Challenge difficulty and settings
- `products`: Catalog configuration

## Important Notes for Development

1. **Intentional Vulnerabilities**: This codebase contains intentional security flaws. Do not replicate these patterns elsewhere.
2. **TypeScript**: All backend code is TypeScript. Compile with `npm run build:server` before running `npm start`.
3. **Frontend Changes**: Frontend is Angular-based. Changes require rebuild or run `npm run serve` for hot reload.
4. **Database Reset**: Delete `data/juiceshop.sqlite` to reset the database. It will be recreated on next startup.
5. **Challenge Development**:

   - Add challenge metadata to `data/static/challenges.yml`
   - Implement vulnerability in appropriate route
   - Add e2e test in `test/e2e/` (Cypress)
   - Document solution in companion guide
6. **Code Style**: ESLint must pass. Run `npm run lint:fix` before committing.
7. **Commit Signing**: All commits must be signed off (Developer Certificate of Origin).

## Context Preservation Files

This repository includes context preservation files in `/docs`:

- `CURRENT_STATE.txt` - Project status snapshot
- `SESSION_HANDOFF.txt` - Session-to-session continuity
- `COMMANDS_LOG.txt` - Command reference library
- `DECISIONS.txt` - Technical decision log
- `ISSUES.txt` - Known issues tracking

These files help maintain project state across Claude Code sessions.

## Node.js Version Compatibility

Officially supported: Node.js 20.x, 22.x, 24.x

- Use the latest minor version of supported major versions
- Docker images use Node.js 22

## Resources

- Official Site: https://owasp-juice.shop
- Documentation: https://pwning.owasp-juice.shop
- GitHub: https://github.com/juice-shop/juice-shop
- Gitter Chat: https://gitter.im/bkimminich/juice-shop
