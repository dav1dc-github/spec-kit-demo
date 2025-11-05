# Phase 0: Research & Technology Decisions

**Feature**: Kanban Task Management Application  
**Date**: 2025-11-05  
**Purpose**: Document technology choices, best practices, and architectural decisions for implementation

## Technology Stack Decisions

### Frontend Framework: React 18 with TypeScript

**Decision**: Use React 18 with TypeScript for the frontend application

**Rationale**:
- React provides mature ecosystem for building interactive UIs with strong community support
- TypeScript ensures type safety across the entire frontend codebase, reducing runtime errors
- React 18's concurrent features enable better performance for real-time updates
- Extensive library support for drag-and-drop (React DnD, dnd-kit) and state management
- Strong developer tooling and documentation

**Alternatives Considered**:
- Vue 3: Simpler learning curve but smaller ecosystem for drag-and-drop libraries
- Svelte: Better performance but less mature real-time collaboration patterns
- Angular: More opinionated structure but heavier bundle size and steeper learning curve

**Best Practices**:
- Use functional components with hooks exclusively (no class components)
- Implement proper memo/useMemo/useCallback for performance optimization
- Follow React Query patterns for server state management
- Use code splitting and lazy loading for route-based optimization
- Implement error boundaries for graceful error handling

### Backend Framework: Node.js with Express

**Decision**: Use Node.js 20 LTS with Express 4 for the backend API

**Rationale**:
- JavaScript/TypeScript consistency across full stack reduces context switching
- Node.js event loop naturally supports high-concurrency real-time features
- Express is lightweight, well-documented, and has extensive middleware ecosystem
- Socket.io integrates seamlessly with Node.js/Express for WebSocket support
- NPM ecosystem provides mature solutions for authentication, validation, and security

**Alternatives Considered**:
- Fastify: Better performance but less mature Socket.io integration patterns
- NestJS: More opinionated structure adds complexity for this project scope
- Python (FastAPI): Requires separate language/ecosystem, no type sharing with frontend

**Best Practices**:
- Use middleware pattern for cross-cutting concerns (logging, auth, validation)
- Implement proper error handling middleware with consistent error responses
- Use async/await consistently, avoid callback hell
- Separate route handlers (controllers) from business logic (services)
- Implement request validation using express-validator or Zod

### Database: PostgreSQL 15+ with Prisma ORM

**Decision**: Use PostgreSQL as the primary database with Prisma ORM for data access

**Rationale**:
- PostgreSQL provides ACID compliance, robust indexing, and excellent performance
- Prisma offers type-safe database queries auto-generated from schema
- Prisma migrations provide version-controlled database schema evolution
- Native support for complex relationships (boards, tasks, users, comments)
- Strong JSON support for flexible metadata storage if needed

**Alternatives Considered**:
- MongoDB: Schema flexibility unnecessary; relational data model is clearer for this domain
- MySQL: Less advanced features for JSON, arrays, and full-text search compared to PostgreSQL
- TypeORM: Less type-safe than Prisma, manual type definitions required

**Best Practices**:
- Index foreign keys (board_id, user_id, task_id) for query performance
- Use Prisma relations for type-safe joins and eager/lazy loading
- Implement database migrations for all schema changes (no direct SQL DDL)
- Use transactions for multi-table operations (e.g., creating board + columns together)
- Implement soft deletes for tasks/comments to maintain audit trail

### Real-Time Communication: Socket.io

**Decision**: Use Socket.io for bidirectional real-time communication

**Rationale**:
- WebSocket with fallback to long-polling ensures broad browser compatibility
- Built-in rooms/namespaces perfect for board-based broadcasting
- Event-based API aligns naturally with user actions (task moved, comment added)
- Automatic reconnection handling reduces client-side complexity
- Strong TypeScript support via typed event maps

**Alternatives Considered**:
- Native WebSocket: Requires manual reconnection logic and fallback handling
- Server-Sent Events: Unidirectional only, requires separate HTTP for client→server
- GraphQL Subscriptions: Adds complexity; overkill for event broadcasting patterns

**Best Practices**:
- Implement authentication middleware for Socket.io connections (JWT verification)
- Use room-based broadcasting (one room per board ID) to limit event scope
- Implement event throttling/debouncing to prevent flooding during rapid changes
- Define strict event schemas with TypeScript for type safety
- Handle disconnections gracefully with automatic rejoin to board rooms
- Implement optimistic UI updates with rollback on server rejection

### State Management: Zustand + React Query

**Decision**: Use Zustand for client state and React Query for server state

**Rationale**:
- Zustand provides lightweight, hook-based state management without boilerplate
- React Query handles server state caching, refetching, and synchronization automatically
- Clear separation between client state (UI) and server state (data) reduces complexity
- React Query's optimistic updates integrate well with real-time features
- Minimal bundle size impact compared to Redux

**Alternatives Considered**:
- Redux Toolkit: More boilerplate, overkill for this application size
- Context API only: Performance issues with frequent updates, no caching
- Jotai/Recoil: Atomic state model unnecessary for this use case

**Best Practices**:
- Use React Query for all API calls and real-time sync with server
- Use Zustand for UI state (modals, selected task, filter preferences)
- Implement optimistic updates via React Query's mutation callbacks
- Configure stale time and cache invalidation strategies per entity type
- Use query keys that include board_id for proper cache segmentation

### Authentication: JWT with httpOnly Cookies

**Decision**: Use JSON Web Tokens stored in httpOnly cookies for authentication

**Rationale**:
- httpOnly cookies prevent XSS attacks (JavaScript cannot access tokens)
- JWT allows stateless authentication across multiple backend instances
- Includes user claims (user_id, email) for authorization without database lookups
- Compatible with both HTTP and WebSocket authentication
- Standard approach with well-documented security patterns

**Alternatives Considered**:
- Session cookies with server-side storage: Requires session store (Redis), adds complexity
- Local storage JWT: Vulnerable to XSS attacks
- OAuth2 only: Adds external dependency for MVP, can be added later

**Best Practices**:
- Use bcrypt with 12 rounds for password hashing
- Set secure and sameSite flags on cookies
- Implement token refresh mechanism (short-lived access tokens, long-lived refresh tokens)
- Include CSRF protection for state-changing operations
- Validate JWT signature and expiration on every authenticated request

### Drag-and-Drop: React DnD or dnd-kit

**Decision**: Use dnd-kit for drag-and-drop functionality

**Rationale**:
- Modern, performant library built specifically for React
- Built-in accessibility features (keyboard navigation, screen reader support)
- Smaller bundle size compared to React DnD
- Better touch device support for mobile/tablet
- Active maintenance and good TypeScript support

**Alternatives Considered**:
- React DnD: More mature but larger bundle, HTML5 backend limitations on touch devices
- Native HTML5 drag-and-drop: Poor mobile support, inconsistent browser behavior
- Manual implementation: Reinventing wheel, accessibility challenges

**Best Practices**:
- Implement keyboard navigation for accessibility (arrow keys to move tasks)
- Provide visual feedback during drag (preview, drop zone highlights)
- Use optimistic updates with rollback on conflict detection
- Throttle position updates during drag to reduce event frequency
- Test extensively on touch devices (iPad, Android tablets)

## Architectural Patterns

### API Design: RESTful with Resource-Oriented URLs

**Pattern**: REST API with standard HTTP methods and resource-based routing

**Rationale**:
- Clear, predictable URL structure (/boards/:boardId/tasks/:taskId)
- Standard HTTP verbs (GET, POST, PUT, DELETE) map to CRUD operations
- Cacheable responses using HTTP cache headers
- Well-understood by all developers, extensive tooling support

**Implementation**:
- Use plural resource names (/boards, /tasks, /comments)
- Nest resources appropriately (/boards/:boardId/tasks)
- Return consistent error response format (status code, message, details)
- Implement pagination for list endpoints (limit, offset, cursor)
- Use proper HTTP status codes (200, 201, 400, 401, 403, 404, 500)

### Real-Time Sync Strategy: Optimistic Updates with Event Sourcing

**Pattern**: Optimistic UI updates with server validation and conflict resolution

**Rationale**:
- Immediate visual feedback improves perceived performance
- Server remains source of truth for conflict resolution
- Event-based broadcasting scales better than polling
- Natural fit for collaborative editing scenarios

**Implementation**:
- Client applies update optimistically (local state change)
- Client emits WebSocket event to server
- Server validates, persists, and broadcasts to all board members
- Clients receive broadcast and reconcile with local state
- On conflict, server version wins; client rolls back optimistic change
- Use version numbers or timestamps for conflict detection

### Error Handling: Centralized with User-Friendly Messages

**Pattern**: Centralized error handling middleware with consistent error format

**Implementation**:
- Backend: Express error middleware catches all errors, formats response
- Frontend: Axios interceptor catches errors, displays user-friendly messages
- Log technical details server-side only (stack traces, SQL errors)
- Show actionable messages to users ("Board not found. It may have been deleted.")
- Implement retry logic for transient failures (network issues)

## Security Considerations

### Input Validation and Sanitization

**Approach**:
- Validate all inputs using express-validator or Zod schemas
- Sanitize HTML content in task descriptions and comments (prevent XSS)
- Enforce length limits (task title: 200 chars, description: 5000 chars)
- Validate email format, password strength requirements
- Use parameterized queries via Prisma (prevents SQL injection)

### Authorization Model: Row-Level Security

**Approach**:
- Check board membership before allowing access to any board resource
- Implement middleware that verifies user has access to requested board
- Use Prisma middleware to automatically filter queries by user access
- Enforce principle of least privilege (board owner vs member permissions)

### Rate Limiting

**Approach**:
- Implement rate limiting on authentication endpoints (5 attempts/15 minutes)
- Rate limit API endpoints per user (100 requests/minute)
- Throttle WebSocket events (max 50 events/second per connection)
- Use express-rate-limit or equivalent middleware

## Performance Optimization

### Database Indexing Strategy

**Indexes to Create**:
- `boards.owner_id` - Find all boards owned by user
- `board_members.user_id` - Find all boards accessible to user  
- `tasks.board_id, tasks.column_id` - List tasks in a column
- `tasks.board_id, tasks.position` - Sort tasks by position
- `comments.task_id, comments.created_at` - List comments chronologically
- Composite index on `board_members(user_id, board_id)` for membership checks

### Frontend Performance

**Strategies**:
- Code splitting: Lazy load Board view, Dashboard separately
- Virtualization: Use react-window for boards with 100+ tasks
- Memoization: Wrap expensive components with React.memo
- Debounce: Debounce search input (300ms delay)
- Image optimization: Use WebP with fallbacks, lazy load avatars

### Caching Strategy

**Approach**:
- React Query: Cache boards for 5 minutes, tasks for 2 minutes
- HTTP: Set cache headers for static assets (immutable)
- Database: Use Prisma query result caching for repeated queries
- WebSocket: Don't cache; always push latest state

## Testing Strategy

### Test Pyramid

**Unit Tests (60%)**:
- Service layer functions (createTask, moveTask, addComment)
- Utility functions (validation, formatting, date handling)
- React hooks (useBoard, useTasks, useAuth)
- Pure components (Button, Card, Input)

**Integration Tests (30%)**:
- API endpoints with test database (POST /boards, GET /tasks)
- WebSocket event handlers (task:move, comment:add)
- Authentication flow (register, login, JWT validation)
- Authorization checks (board access, task permissions)

**E2E Tests (10%)**:
- P1: Basic Task Management (create board, add tasks, drag-and-drop)
- P2: Real-Time Collaboration (two users, verify sync)
- Critical path: Login → Dashboard → Board → Create Task → Comment

### Test Data Strategy

**Approach**:
- Use separate test database (Prisma test environment)
- Seed test database with factories (board factory, user factory)
- Clean up after each test (transaction rollback or database reset)
- Use deterministic test data (avoid random values in assertions)

## Deployment Considerations

### Environment Configuration

**Environments**:
- Development: Local PostgreSQL, localhost:3000 (frontend), localhost:5000 (backend)
- Staging: Cloud PostgreSQL, HTTPS, production-like configuration
- Production: Managed PostgreSQL, CDN for frontend, load-balanced backend

**Environment Variables**:
- `DATABASE_URL`: PostgreSQL connection string
- `JWT_SECRET`: Token signing key (min 32 characters)
- `JWT_EXPIRES_IN`: Token expiration (15m access, 7d refresh)
- `CORS_ORIGIN`: Allowed frontend origin(s)
- `NODE_ENV`: development | staging | production

### Monitoring and Observability

**Metrics to Track**:
- API response times (p50, p95, p99)
- WebSocket connection count, message throughput
- Database query performance, connection pool usage
- Error rates by endpoint and error type
- User activity (boards created, tasks moved, comments added)

**Tools**:
- Application: Prometheus + Grafana (metrics)
- Errors: Sentry (error tracking and reporting)
- Logs: Structured logging with Winston, aggregated in CloudWatch or ELK
- Database: PostgreSQL slow query log, pg_stat_statements

## Open Questions and Risks

### Risks Identified

1. **Real-Time Sync Scalability**: 10 concurrent users per board is specified; needs load testing to validate
2. **Conflict Resolution**: Last-write-wins may not be acceptable for all scenarios; may need CRDTs for complex conflicts
3. **Mobile Experience**: Touch-based drag-and-drop requires careful testing; may need native mobile apps eventually
4. **Offline Support**: Spec mentions reconnection handling but not offline-first; may need service workers for true offline mode

### Recommended Mitigations

1. Load test with 10+ concurrent users early to validate architecture
2. Start with last-write-wins; monitor conflict frequency to inform CRDT investment decision
3. Prioritize mobile browser testing in P1; defer native apps to post-MVP
4. Document online-only limitation; consider offline support as separate P5 user story

## Next Steps

1. ✅ Phase 0 Complete: Technology decisions documented
2. → Phase 1: Design data model and API contracts
3. → Phase 1: Create quickstart guide for development setup
4. → Phase 1: Update agent context with chosen technologies
5. → Phase 2: Generate task breakdown (via `/speckit.tasks`)
