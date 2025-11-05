# Quickstart Guide: Kanban Task Management Application

**Feature**: Kanban Task Management Application  
**Purpose**: Get development environment up and running quickly

## Prerequisites

Ensure you have the following installed:

- **Node.js**: v20 LTS or higher ([Download](https://nodejs.org/))
- **npm**: v10 or higher (comes with Node.js)
- **PostgreSQL**: v15 or higher ([Download](https://www.postgresql.org/download/))
- **Git**: Latest version
- **VS Code** (recommended): With extensions:
  - ESLint
  - Prettier
  - Prisma
  - Thunder Client or REST Client (for API testing)

## Quick Setup (5 Minutes)

### 1. Clone and Install Dependencies

```bash
# Navigate to project root (if not already there)
cd /path/to/spec-kit-demo

# Install backend dependencies
cd backend
npm install

# Install frontend dependencies
cd ../frontend
npm install
```

### 2. Configure Database

```bash
# Create PostgreSQL database
psql -U postgres
CREATE DATABASE kanban_dev;
\q

# Or use command line directly
createdb -U postgres kanban_dev
```

### 3. Setup Environment Variables

**Backend** (`backend/.env`):

```env
# Database
DATABASE_URL="postgresql://postgres:password@localhost:5432/kanban_dev?schema=public"

# JWT
JWT_SECRET="your-super-secret-jwt-key-min-32-characters-long"
JWT_EXPIRES_IN="15m"
JWT_REFRESH_EXPIRES_IN="7d"

# Server
PORT=5000
NODE_ENV="development"

# CORS
CORS_ORIGIN="http://localhost:3000"

# WebSocket
WEBSOCKET_CORS_ORIGIN="http://localhost:3000"
```

**Frontend** (`frontend/.env`):

```env
VITE_API_URL=http://localhost:5000/api/v1
VITE_WS_URL=http://localhost:5000
```

### 4. Initialize Database

```bash
# From backend/ directory
cd backend

# Generate Prisma client
npx prisma generate

# Run database migrations
npx prisma migrate dev --name init

# (Optional) Seed database with sample data
npm run seed
```

### 5. Start Development Servers

**Terminal 1 - Backend**:
```bash
cd backend
npm run dev
```

**Terminal 2 - Frontend**:
```bash
cd frontend
npm run dev
```

### 6. Verify Setup

- Frontend: [http://localhost:3000](http://localhost:3000)
- Backend API: [http://localhost:5000/api/v1](http://localhost:5000/api/v1)
- Prisma Studio (database GUI): `npx prisma studio` (from backend/)

## Project Structure

```
spec-kit-demo/
├── backend/
│   ├── src/
│   │   ├── config/          # App configuration
│   │   ├── controllers/     # HTTP request handlers
│   │   ├── services/        # Business logic
│   │   ├── middleware/      # Express middleware
│   │   ├── validators/      # Input validation
│   │   ├── websocket/       # Socket.io handlers
│   │   ├── utils/           # Shared utilities
│   │   └── server.ts        # Entry point
│   ├── tests/               # Test files
│   ├── prisma/              # Database schema & migrations
│   └── package.json
│
├── frontend/
│   ├── src/
│   │   ├── components/      # React components
│   │   ├── pages/           # Page components
│   │   ├── hooks/           # Custom hooks
│   │   ├── services/        # API & WebSocket clients
│   │   ├── store/           # Zustand stores
│   │   ├── types/           # TypeScript types
│   │   └── main.tsx         # Entry point
│   ├── tests/               # Test files
│   └── package.json
│
└── specs/                   # Documentation
    └── 001-kanban-task-manager/
        ├── spec.md
        ├── plan.md
        ├── research.md
        ├── data-model.md
        ├── contracts/
        └── quickstart.md    # This file
```

## Development Workflow

### Backend Development

#### Run Development Server

```bash
cd backend
npm run dev  # Hot-reload with nodemon
```

#### Run Tests

```bash
# Unit tests
npm run test:unit

# Integration tests
npm run test:integration

# All tests
npm test

# Coverage report
npm run test:coverage
```

#### Database Commands

```bash
# Create new migration
npx prisma migrate dev --name <migration_name>

# Reset database (destructive!)
npx prisma migrate reset

# Open Prisma Studio
npx prisma studio

# Format schema
npx prisma format
```

#### API Testing

Use Thunder Client, Postman, or curl:

```bash
# Register user
curl -X POST http://localhost:5000/api/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","username":"testuser","password":"password123"}'

# Login
curl -X POST http://localhost:5000/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"password123"}' \
  -c cookies.txt

# Create board (uses cookie from login)
curl -X POST http://localhost:5000/api/v1/boards \
  -H "Content-Type: application/json" \
  -b cookies.txt \
  -d '{"name":"My First Board","description":"Testing board creation"}'
```

### Frontend Development

#### Run Development Server

```bash
cd frontend
npm run dev  # Vite dev server with HMR
```

#### Run Tests

```bash
# Unit tests
npm run test:unit

# E2E tests
npm run test:e2e

# All tests
npm test
```

#### Build for Production

```bash
npm run build   # Build to dist/
npm run preview # Preview production build
```

### Code Quality

#### Linting & Formatting

```bash
# Backend
cd backend
npm run lint        # Run ESLint
npm run lint:fix    # Auto-fix issues
npm run format      # Run Prettier

# Frontend
cd frontend
npm run lint
npm run lint:fix
npm run format
```

#### Type Checking

```bash
# Backend
npm run type-check

# Frontend
npm run type-check
```

## Common Development Tasks

### Creating a New Feature

1. **Update Prisma Schema** (if database changes needed):
   ```bash
   cd backend/prisma
   # Edit schema.prisma
   npx prisma migrate dev --name add_<feature>
   ```

2. **Generate API Endpoint**:
   - Create controller in `backend/src/controllers/`
   - Create service in `backend/src/services/`
   - Add route in `backend/src/routes/`
   - Add validator in `backend/src/validators/`

3. **Write Tests First** (TDD):
   ```bash
   # Create test file
   touch backend/tests/integration/<feature>.test.ts
   # Write tests
   # Run tests (should fail)
   npm test
   # Implement feature
   # Run tests (should pass)
   npm test
   ```

4. **Add Frontend Component**:
   - Create component in `frontend/src/components/`
   - Create hook if needed in `frontend/src/hooks/`
   - Update state management in `frontend/src/store/`

### Adding Real-Time Features

1. **Define Event in Backend**:
   ```typescript
   // backend/src/websocket/events/<feature>.events.ts
   export const handleFeatureEvent = (socket, io, data) => {
     // Validate, process, broadcast
     socket.to(`board:${data.boardId}`).emit('feature:updated', result);
   };
   ```

2. **Listen in Frontend**:
   ```typescript
   // frontend/src/hooks/useWebSocket.ts
   useEffect(() => {
     socket.on('feature:updated', (data) => {
       // Update local state
     });
     return () => socket.off('feature:updated');
   }, [socket]);
   ```

### Debugging

#### Backend Debugging

```bash
# Enable debug logs
DEBUG=* npm run dev

# Or specific namespace
DEBUG=app:* npm run dev
```

With VS Code, add launch configuration (`.vscode/launch.json`):

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Debug Backend",
      "runtimeExecutable": "npm",
      "runtimeArgs": ["run", "dev"],
      "cwd": "${workspaceFolder}/backend",
      "console": "integratedTerminal",
      "internalConsoleOptions": "neverOpen"
    }
  ]
}
```

#### Frontend Debugging

Use browser DevTools:
- React DevTools extension
- Network tab for API calls
- Console for errors and logs

## Testing Guidelines

### Test Structure (TDD)

**1. Write Test First**:
```typescript
describe('Board Service', () => {
  it('should create a new board', async () => {
    const boardData = { name: 'Test Board', ownerId: userId };
    const board = await boardService.createBoard(boardData);
    
    expect(board).toBeDefined();
    expect(board.name).toBe('Test Board');
  });
});
```

**2. Run Test (Should Fail)**:
```bash
npm test -- board.service.test.ts
```

**3. Implement Minimum Code**:
```typescript
// src/services/board.service.ts
export async function createBoard(data) {
  return prisma.board.create({ data });
}
```

**4. Run Test (Should Pass)**:
```bash
npm test -- board.service.test.ts
```

**5. Refactor**:
```typescript
// Add validation, error handling, etc.
```

### Test Database

Tests use a separate test database:

```bash
# Setup (one-time)
createdb -U postgres kanban_test

# Set TEST_DATABASE_URL in .env.test
TEST_DATABASE_URL="postgresql://postgres:password@localhost:5432/kanban_test"

# Migrations run automatically before tests
npm test
```

## Troubleshooting

### Database Connection Issues

```bash
# Check PostgreSQL is running
pg_isready

# Restart PostgreSQL
brew services restart postgresql  # macOS
sudo systemctl restart postgresql # Linux

# Verify connection string in .env
# Check username, password, database name
```

### Port Already in Use

```bash
# Find process using port 5000
lsof -i :5000

# Kill process
kill -9 <PID>

# Or use different port in .env
PORT=5001
```

### Prisma Schema Drift

```bash
# Reset database and migrations
npx prisma migrate reset

# Or generate new migration
npx prisma migrate dev
```

### WebSocket Connection Failed

- Verify `VITE_WS_URL` in frontend `.env`
- Check CORS settings in backend `websocket.config.ts`
- Ensure backend server is running
- Check browser console for connection errors

### Type Errors

```bash
# Regenerate Prisma client
cd backend
npx prisma generate

# Rebuild frontend
cd frontend
rm -rf node_modules dist
npm install
```

## Performance Tips

### Development Mode

- Use browser dev tools Performance tab
- Enable React DevTools Profiler
- Monitor Network tab for slow API calls
- Check database query performance with `npx prisma studio`

### Database Optimization

```bash
# Analyze slow queries
psql kanban_dev

# Show slow queries
SELECT * FROM pg_stat_statements ORDER BY total_time DESC LIMIT 10;

# Add indexes as needed (update schema.prisma)
```

## Next Steps

1. ✅ Development environment ready
2. → Read [spec.md](./spec.md) for feature requirements
3. → Review [data-model.md](./data-model.md) for database schema
4. → Check [contracts/](./contracts/) for API and WebSocket contracts
5. → Run `/speckit.tasks` to generate task breakdown
6. → Start implementing User Story 1 (Basic Task Management) following TDD

## Useful Commands Reference

```bash
# Backend
npm run dev              # Start dev server
npm test                 # Run all tests
npm run lint            # Run linter
npx prisma studio       # Open DB GUI
npx prisma migrate dev  # Create migration

# Frontend
npm run dev             # Start dev server
npm test                # Run tests
npm run build           # Production build
npm run lint            # Run linter

# Database
createdb kanban_dev               # Create database
dropdb kanban_dev                 # Drop database
psql kanban_dev                   # Connect to database
pg_dump kanban_dev > backup.sql  # Backup database
psql kanban_dev < backup.sql     # Restore database
```

## Resources

- [React Documentation](https://react.dev)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)
- [Prisma Documentation](https://www.prisma.io/docs)
- [Socket.io Documentation](https://socket.io/docs/)
- [Express.js Guide](https://expressjs.com/en/guide/routing.html)
- [Vitest Documentation](https://vitest.dev)
- [Playwright Documentation](https://playwright.dev)

## Support

For issues or questions:
1. Check this quickstart guide
2. Review [research.md](./research.md) for architectural decisions
3. Consult API contracts in [contracts/](./contracts/)
4. Check constitution for code standards: [.specify/memory/constitution.md](../../.specify/memory/constitution.md)
