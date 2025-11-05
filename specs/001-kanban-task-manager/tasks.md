# Tasks: Kanban Task Management Application

**Input**: Design documents from `/specs/001-kanban-task-manager/`
**Prerequisites**: plan.md (required), spec.md (required), data-model.md, contracts/, research.md, quickstart.md

**Tests**: Per constitution requirement, all tasks follow TDD workflow - tests written before implementation.

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3, US4)
- Include exact file paths in descriptions

## Path Conventions

- **Web app structure**: `backend/src/`, `frontend/src/`
- **Tests**: `backend/tests/`, `frontend/tests/`

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization and basic structure

- [ ] T001 Create project structure with backend/ and frontend/ directories per implementation plan
- [ ] T002 Initialize backend with TypeScript 5.3+, Node.js 20 LTS, Express 4, and Prisma ORM in backend/package.json
- [ ] T003 [P] Initialize frontend with React 18, TypeScript 5.3+, and Vite in frontend/package.json
- [ ] T004 [P] Configure ESLint and Prettier for both backend and frontend with strict TypeScript rules
- [ ] T005 [P] Setup PostgreSQL 15+ database and configure connection string in backend/.env
- [ ] T006 Setup Prisma schema in backend/prisma/schema.prisma with all 6 entities (User, Board, BoardMember, Column, Task, Comment)
- [ ] T007 Generate initial Prisma migration and apply to database
- [ ] T008 [P] Configure Vitest for frontend testing in frontend/vitest.config.ts
- [ ] T009 [P] Configure Jest for backend testing in backend/jest.config.js
- [ ] T010 [P] Setup Tailwind CSS in frontend with design system configuration
- [ ] T011 Setup GitHub Actions workflow for CI/CD with test suites

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core infrastructure that MUST be complete before ANY user story can be implemented

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

- [ ] T012 Implement bcrypt password hashing utility (12 rounds) in backend/src/utils/auth.ts
- [ ] T013 Implement JWT token generation and verification in backend/src/utils/jwt.ts with httpOnly cookie support
- [ ] T014 Create authentication middleware in backend/src/middleware/auth.ts to validate JWT tokens
- [ ] T015 Create authorization middleware in backend/src/middleware/authorize.ts for board access checks
- [ ] T016 [P] Implement centralized error handling middleware in backend/src/middleware/errorHandler.ts
- [ ] T017 [P] Setup express-validator for input validation in backend/src/middleware/validation.ts
- [ ] T018 [P] Configure environment variables management in backend/src/config/env.ts
- [ ] T019 [P] Setup logging with Winston in backend/src/utils/logger.ts
- [ ] T020 Create Prisma client singleton in backend/src/lib/prisma.ts
- [ ] T021 [P] Setup Socket.io server in backend/src/lib/socket.ts with JWT authentication
- [ ] T022 [P] Setup Socket.io client connection in frontend/src/lib/socket.ts with reconnection logic
- [ ] T023 [P] Create base API client with axios in frontend/src/lib/api.ts
- [ ] T024 [P] Setup React Query client in frontend/src/lib/queryClient.ts
- [ ] T025 [P] Setup Zustand store for global state in frontend/src/store/index.ts
- [ ] T026 [P] Create reusable UI components library in frontend/src/components/ui/ (Button, Input, Card, Modal)
- [ ] T027 [P] Configure React Router in frontend/src/App.tsx with protected routes

**Checkpoint**: Foundation ready - user story implementation can now begin in parallel

---

## Phase 3: User Story 1 - Basic Task Management (Priority: P1) 🎯 MVP

**Goal**: Users can create, view, organize, and edit tasks in a visual Kanban board

**Independent Test**: Create tasks, move them between columns, edit task details, verify persistence after page refresh

### Tests for User Story 1 (TDD Red Phase)

> **TDD REQUIREMENT**: Write these tests FIRST, ensure they FAIL before implementation

- [ ] T028 [P] [US1] Write backend integration test for POST /api/v1/auth/register endpoint in backend/tests/integration/auth.test.ts
- [ ] T029 [P] [US1] Write backend integration test for POST /api/v1/auth/login endpoint in backend/tests/integration/auth.test.ts
- [ ] T030 [P] [US1] Write backend integration test for POST /api/v1/boards (create board) in backend/tests/integration/boards.test.ts
- [ ] T031 [P] [US1] Write backend integration test for POST /api/v1/columns (create column) in backend/tests/integration/columns.test.ts
- [ ] T032 [P] [US1] Write backend integration test for POST /api/v1/tasks (create task) in backend/tests/integration/tasks.test.ts
- [ ] T033 [P] [US1] Write backend integration test for PUT /api/v1/tasks/:taskId/move (move task) in backend/tests/integration/tasks.test.ts
- [ ] T034 [P] [US1] Write frontend component test for TaskCard drag-and-drop in frontend/tests/components/TaskCard.test.tsx
- [ ] T035 [P] [US1] Write E2E test for complete task creation and organization flow in frontend/tests/e2e/basic-task-management.spec.ts using Playwright

### Implementation for User Story 1 (TDD Green Phase)

#### Backend API Endpoints

- [ ] T036 [P] [US1] Implement POST /api/v1/auth/register endpoint in backend/src/routes/auth.ts with email/username/password validation
- [ ] T037 [P] [US1] Implement POST /api/v1/auth/login endpoint in backend/src/routes/auth.ts with JWT cookie creation
- [ ] T038 [P] [US1] Implement POST /api/v1/auth/logout endpoint in backend/src/routes/auth.ts to clear JWT cookie
- [ ] T039 [P] [US1] Implement GET /api/v1/auth/me endpoint in backend/src/routes/auth.ts to return current user
- [ ] T040 [US1] Implement User service layer in backend/src/services/userService.ts with password hashing and validation
- [ ] T041 [P] [US1] Implement POST /api/v1/boards endpoint in backend/src/routes/boards.ts to create board with default columns
- [ ] T042 [P] [US1] Implement GET /api/v1/boards endpoint in backend/src/routes/boards.ts to list user's accessible boards
- [ ] T043 [P] [US1] Implement GET /api/v1/boards/:boardId endpoint in backend/src/routes/boards.ts with columns and tasks
- [ ] T044 [P] [US1] Implement PUT /api/v1/boards/:boardId endpoint in backend/src/routes/boards.ts to update board details
- [ ] T045 [P] [US1] Implement DELETE /api/v1/boards/:boardId endpoint in backend/src/routes/boards.ts (soft delete with is_archived)
- [ ] T046 [US1] Implement Board service layer in backend/src/services/boardService.ts with authorization checks
- [ ] T047 [P] [US1] Implement POST /api/v1/boards/:boardId/columns endpoint in backend/src/routes/columns.ts to create column
- [ ] T048 [P] [US1] Implement PUT /api/v1/boards/:boardId/columns/:columnId endpoint in backend/src/routes/columns.ts to update column
- [ ] T049 [P] [US1] Implement DELETE /api/v1/boards/:boardId/columns/:columnId endpoint in backend/src/routes/columns.ts
- [ ] T050 [US1] Implement Column service layer in backend/src/services/columnService.ts with position management
- [ ] T051 [P] [US1] Implement POST /api/v1/boards/:boardId/tasks endpoint in backend/src/routes/tasks.ts to create task
- [ ] T052 [P] [US1] Implement GET /api/v1/boards/:boardId/tasks/:taskId endpoint in backend/src/routes/tasks.ts to get task details
- [ ] T053 [P] [US1] Implement PUT /api/v1/boards/:boardId/tasks/:taskId endpoint in backend/src/routes/tasks.ts to update task
- [ ] T054 [P] [US1] Implement DELETE /api/v1/boards/:boardId/tasks/:taskId endpoint in backend/src/routes/tasks.ts
- [ ] T055 [P] [US1] Implement PUT /api/v1/boards/:boardId/tasks/:taskId/move endpoint in backend/src/routes/tasks.ts to move task between columns
- [ ] T056 [US1] Implement Task service layer in backend/src/services/taskService.ts with position recalculation logic

#### Frontend Components & Pages

- [ ] T057 [P] [US1] Create LoginPage component in frontend/src/pages/LoginPage.tsx with form validation
- [ ] T058 [P] [US1] Create RegisterPage component in frontend/src/pages/RegisterPage.tsx with form validation
- [ ] T059 [P] [US1] Create DashboardPage component in frontend/src/pages/DashboardPage.tsx to list user's boards
- [ ] T060 [P] [US1] Create BoardPage component in frontend/src/pages/BoardPage.tsx as main Kanban board view
- [ ] T061 [P] [US1] Create TaskCard component in frontend/src/components/TaskCard.tsx with drag-and-drop using dnd-kit
- [ ] T062 [P] [US1] Create Column component in frontend/src/components/Column.tsx to display tasks in vertical list
- [ ] T063 [P] [US1] Create TaskDetailModal component in frontend/src/components/TaskDetailModal.tsx for editing task details
- [ ] T064 [P] [US1] Create CreateTaskForm component in frontend/src/components/CreateTaskForm.tsx with title/description/priority/due date
- [ ] T065 [P] [US1] Create CreateBoardModal component in frontend/src/components/CreateBoardModal.tsx
- [ ] T066 [US1] Implement useAuth custom hook in frontend/src/hooks/useAuth.ts for authentication state management
- [ ] T067 [US1] Implement React Query hooks for boards in frontend/src/hooks/useBoards.ts (useBoards, useBoard, useCreateBoard)
- [ ] T068 [US1] Implement React Query hooks for tasks in frontend/src/hooks/useTasks.ts (useCreateTask, useUpdateTask, useMoveTask, useDeleteTask)
- [ ] T069 [US1] Setup Zustand auth store in frontend/src/store/authStore.ts for user session state
- [ ] T070 [US1] Implement drag-and-drop logic with dnd-kit in frontend/src/components/BoardPage.tsx with optimistic updates

**Checkpoint**: At this point, User Story 1 should be fully functional and testable independently - users can register, login, create boards, create tasks, and organize tasks with drag-and-drop

---

## Phase 4: User Story 2 - Real-Time Collaboration (Priority: P2)

**Goal**: Multiple users see each other's changes instantly without manual refresh

**Independent Test**: Open same board in two browser windows, make changes in one, verify they appear in the other within 2 seconds

### Tests for User Story 2 (TDD Red Phase)

> **TDD REQUIREMENT**: Write these tests FIRST, ensure they FAIL before implementation

- [ ] T071 [P] [US2] Write Socket.io contract test for board:join event in backend/tests/socket/board.events.test.ts
- [ ] T072 [P] [US2] Write Socket.io contract test for task:create event broadcast in backend/tests/socket/task.events.test.ts
- [ ] T073 [P] [US2] Write Socket.io contract test for task:move event broadcast in backend/tests/socket/task.events.test.ts
- [ ] T074 [P] [US2] Write E2E test for real-time task creation synchronization in frontend/tests/e2e/real-time-collaboration.spec.ts using two browser contexts

### Implementation for User Story 2 (TDD Green Phase)

#### Backend WebSocket Events

- [ ] T075 [P] [US2] Implement board:join event handler in backend/src/socket/handlers/boardHandlers.ts to join board room
- [ ] T076 [P] [US2] Implement board:leave event handler in backend/src/socket/handlers/boardHandlers.ts to leave board room
- [ ] T077 [P] [US2] Implement board:update event handler in backend/src/socket/handlers/boardHandlers.ts with broadcast
- [ ] T078 [P] [US2] Implement column:create event handler in backend/src/socket/handlers/columnHandlers.ts with broadcast
- [ ] T079 [P] [US2] Implement column:update event handler in backend/src/socket/handlers/columnHandlers.ts with broadcast
- [ ] T080 [P] [US2] Implement column:delete event handler in backend/src/socket/handlers/columnHandlers.ts with broadcast
- [ ] T081 [P] [US2] Implement task:create event handler in backend/src/socket/handlers/taskHandlers.ts with broadcast
- [ ] T082 [P] [US2] Implement task:update event handler in backend/src/socket/handlers/taskHandlers.ts with broadcast
- [ ] T083 [P] [US2] Implement task:move event handler in backend/src/socket/handlers/taskHandlers.ts with broadcast and position recalculation
- [ ] T084 [P] [US2] Implement task:delete event handler in backend/src/socket/handlers/taskHandlers.ts with broadcast
- [ ] T085 [P] [US2] Implement user:editing event handler in backend/src/socket/handlers/presenceHandlers.ts to broadcast editing indicators
- [ ] T086 [P] [US2] Implement user:stopped-editing event handler in backend/src/socket/handlers/presenceHandlers.ts
- [ ] T087 [US2] Add conflict resolution logic for concurrent task moves in backend/src/services/taskService.ts using last-write-wins

#### Frontend Real-Time Integration

- [ ] T088 [P] [US2] Create useSocketBoard hook in frontend/src/hooks/useSocketBoard.ts to listen for board events
- [ ] T089 [P] [US2] Create useSocketTasks hook in frontend/src/hooks/useSocketTasks.ts to listen for task events
- [ ] T090 [US2] Integrate board:updated event listener in frontend/src/pages/BoardPage.tsx with React Query cache updates
- [ ] T091 [US2] Integrate task:create event listener in frontend/src/pages/BoardPage.tsx with optimistic UI updates
- [ ] T092 [US2] Integrate task:update event listener in frontend/src/components/TaskCard.tsx with React Query cache invalidation
- [ ] T093 [US2] Integrate task:move event listener in frontend/src/pages/BoardPage.tsx with dnd-kit state updates
- [ ] T094 [US2] Integrate task:delete event listener in frontend/src/pages/BoardPage.tsx with React Query cache removal
- [ ] T095 [P] [US2] Add ActiveUsersIndicator component in frontend/src/components/ActiveUsersIndicator.tsx showing users on board
- [ ] T096 [P] [US2] Add EditingIndicator component in frontend/src/components/EditingIndicator.tsx showing who's editing tasks
- [ ] T097 [US2] Emit Socket.io events on user actions (create/update/move tasks) in frontend/src/hooks/useTasks.ts
- [ ] T098 [US2] Handle WebSocket reconnection with state resynchronization in frontend/src/lib/socket.ts

**Checkpoint**: At this point, User Stories 1 AND 2 should both work - single-user task management + real-time multi-user synchronization

---

## Phase 5: User Story 3 - Task Comments and Discussion (Priority: P3)

**Goal**: Team members can discuss tasks, share updates, and ask questions directly on task cards

**Independent Test**: Add comments to a task, verify persistence, confirm multiple users see comments in real-time

### Tests for User Story 3 (TDD Red Phase)

> **TDD REQUIREMENT**: Write these tests FIRST, ensure they FAIL before implementation

- [ ] T099 [P] [US3] Write backend integration test for POST /api/v1/tasks/:taskId/comments in backend/tests/integration/comments.test.ts
- [ ] T100 [P] [US3] Write backend integration test for GET /api/v1/tasks/:taskId/comments in backend/tests/integration/comments.test.ts
- [ ] T101 [P] [US3] Write Socket.io contract test for comment:create event broadcast in backend/tests/socket/comment.events.test.ts
- [ ] T102 [P] [US3] Write E2E test for commenting flow with real-time updates in frontend/tests/e2e/task-comments.spec.ts

### Implementation for User Story 3 (TDD Green Phase)

#### Backend API & WebSocket for Comments

- [ ] T103 [P] [US3] Implement POST /api/v1/tasks/:taskId/comments endpoint in backend/src/routes/comments.ts to create comment
- [ ] T104 [P] [US3] Implement GET /api/v1/tasks/:taskId/comments endpoint in backend/src/routes/comments.ts to list comments
- [ ] T105 [P] [US3] Implement PUT /api/v1/tasks/:taskId/comments/:commentId endpoint in backend/src/routes/comments.ts to edit comment
- [ ] T106 [P] [US3] Implement DELETE /api/v1/tasks/:taskId/comments/:commentId endpoint in backend/src/routes/comments.ts
- [ ] T107 [US3] Implement Comment service layer in backend/src/services/commentService.ts with threading support
- [ ] T108 [P] [US3] Implement comment:create event handler in backend/src/socket/handlers/commentHandlers.ts with broadcast
- [ ] T109 [P] [US3] Implement comment:update event handler in backend/src/socket/handlers/commentHandlers.ts with broadcast
- [ ] T110 [P] [US3] Implement comment:delete event handler in backend/src/socket/handlers/commentHandlers.ts with broadcast

#### Frontend Comments UI

- [ ] T111 [P] [US3] Create CommentList component in frontend/src/components/CommentList.tsx to display comments chronologically
- [ ] T112 [P] [US3] Create CommentItem component in frontend/src/components/CommentItem.tsx with author, timestamp, and threaded replies
- [ ] T113 [P] [US3] Create CommentForm component in frontend/src/components/CommentForm.tsx for posting new comments
- [ ] T114 [US3] Integrate CommentList into TaskDetailModal in frontend/src/components/TaskDetailModal.tsx
- [ ] T115 [US3] Implement React Query hooks for comments in frontend/src/hooks/useComments.ts (useComments, useCreateComment, useUpdateComment, useDeleteComment)
- [ ] T116 [US3] Create useSocketComments hook in frontend/src/hooks/useSocketComments.ts to listen for comment events
- [ ] T117 [US3] Integrate comment:create event listener in frontend/src/components/CommentList.tsx with optimistic updates
- [ ] T118 [US3] Integrate comment:update event listener in frontend/src/components/CommentItem.tsx
- [ ] T119 [US3] Integrate comment:delete event listener in frontend/src/components/CommentList.tsx

**Checkpoint**: All three user stories (1, 2, 3) should now be independently functional - task management + real-time sync + commenting

---

## Phase 6: User Story 4 - Multi-User Workspace Management (Priority: P4)

**Goal**: Organizations can control who can access and modify boards with proper permissions

**Independent Test**: Invite users to boards, verify they can access shared boards, confirm uninvited users cannot access private boards

### Tests for User Story 4 (TDD Red Phase)

> **TDD REQUIREMENT**: Write these tests FIRST, ensure they FAIL before implementation

- [ ] T120 [P] [US4] Write backend integration test for POST /api/v1/boards/:boardId/members (invite user) in backend/tests/integration/boardMembers.test.ts
- [ ] T121 [P] [US4] Write backend integration test for DELETE /api/v1/boards/:boardId/members/:userId (remove user) in backend/tests/integration/boardMembers.test.ts
- [ ] T122 [P] [US4] Write backend integration test for board access authorization in backend/tests/integration/authorization.test.ts
- [ ] T123 [P] [US4] Write E2E test for board invitation and access flow in frontend/tests/e2e/board-sharing.spec.ts

### Implementation for User Story 4 (TDD Green Phase)

#### Backend Board Membership & Authorization

- [ ] T124 [P] [US4] Implement POST /api/v1/boards/:boardId/members endpoint in backend/src/routes/boardMembers.ts to invite user by email
- [ ] T125 [P] [US4] Implement GET /api/v1/boards/:boardId/members endpoint in backend/src/routes/boardMembers.ts to list board members
- [ ] T126 [P] [US4] Implement DELETE /api/v1/boards/:boardId/members/:userId endpoint in backend/src/routes/boardMembers.ts to remove member
- [ ] T127 [US4] Implement BoardMember service layer in backend/src/services/boardMemberService.ts with role management
- [ ] T128 [US4] Enhance authorization middleware in backend/src/middleware/authorize.ts to check BoardMember access
- [ ] T129 [US4] Add board ownership check utility in backend/src/utils/permissions.ts to verify owner-only operations
- [ ] T130 [US4] Update Board service to filter boards by user access in backend/src/services/boardService.ts

#### Frontend Board Sharing UI

- [ ] T131 [P] [US4] Create InviteMemberModal component in frontend/src/components/InviteMemberModal.tsx with email input
- [ ] T132 [P] [US4] Create BoardMembersList component in frontend/src/components/BoardMembersList.tsx showing members with remove option
- [ ] T133 [P] [US4] Create BoardAccessBadge component in frontend/src/components/BoardAccessBadge.tsx indicating ownership/membership
- [ ] T134 [US4] Integrate InviteMemberModal into BoardPage header in frontend/src/pages/BoardPage.tsx
- [ ] T135 [US4] Implement React Query hooks for board members in frontend/src/hooks/useBoardMembers.ts (useBoardMembers, useInviteMember, useRemoveMember)
- [ ] T136 [US4] Update DashboardPage to display board ownership indicators in frontend/src/pages/DashboardPage.tsx
- [ ] T137 [US4] Add access denied error handling in frontend/src/lib/api.ts with redirect to dashboard

**Checkpoint**: All four user stories should now be fully functional - complete collaborative Kanban application with access control

---

## Phase 7: Polish & Cross-Cutting Concerns

**Purpose**: Improvements that affect multiple user stories and production readiness

- [ ] T138 [P] Add task search and filtering endpoint GET /api/v1/boards/:boardId/tasks with query params in backend/src/routes/tasks.ts
- [ ] T139 [P] Add task assignment feature (assign to board members) in backend/src/services/taskService.ts
- [ ] T140 [P] Create SearchBar component in frontend/src/components/SearchBar.tsx for task filtering
- [ ] T141 [P] Create TaskAssigneeSelect component in frontend/src/components/TaskAssigneeSelect.tsx
- [ ] T142 [P] Add loading skeleton components in frontend/src/components/skeletons/ for better UX
- [ ] T143 [P] Implement error boundary component in frontend/src/components/ErrorBoundary.tsx
- [ ] T144 [P] Add rate limiting middleware in backend/src/middleware/rateLimit.ts using express-rate-limit
- [ ] T145 [P] Setup Sentry for error tracking in both backend and frontend
- [ ] T146 [P] Add database indexing validation script in backend/scripts/validate-indexes.ts
- [ ] T147 [P] Create API documentation using Swagger UI in backend/src/routes/docs.ts from OpenAPI spec
- [ ] T148 [P] Add performance monitoring with Prometheus metrics in backend/src/middleware/metrics.ts
- [ ] T149 [P] Implement audit logging for sensitive operations in backend/src/services/auditService.ts
- [ ] T150 [P] Add comprehensive README.md with quickstart instructions
- [ ] T151 [P] Add Docker Compose setup for local development in docker-compose.yml
- [ ] T152 [P] Create deployment guide in docs/deployment.md
- [ ] T153 [P] Add accessibility audit and fixes (WCAG 2.1 Level AA) for all frontend components
- [ ] T154 [P] Optimize bundle size with code splitting in frontend/vite.config.ts
- [ ] T155 [P] Add unit tests for service layer methods in backend/tests/unit/services/
- [ ] T156 [P] Add unit tests for custom hooks in frontend/tests/unit/hooks/
- [ ] T157 Run quickstart.md validation to ensure setup instructions work
- [ ] T158 Security audit with npm audit and dependency updates
- [ ] T159 Performance testing with load testing tool (Artillery or k6)
- [ ] T160 Final code review and refactoring pass

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - can start immediately
- **Foundational (Phase 2)**: Depends on Phase 1 completion - BLOCKS all user stories
- **User Story 1 (Phase 3)**: Depends on Phase 2 completion - MVP functionality
- **User Story 2 (Phase 4)**: Depends on Phase 2 completion AND Phase 3 recommended (builds on task management)
- **User Story 3 (Phase 5)**: Depends on Phase 2 completion AND Phase 3 recommended (comments require tasks)
- **User Story 4 (Phase 6)**: Depends on Phase 2 completion AND Phase 3 recommended (sharing requires boards)
- **Polish (Phase 7)**: Depends on desired user stories being complete (P1-P4 for full feature set)

### User Story Dependencies

- **User Story 1 (P1 - MVP)**: Can start after Foundational - No dependencies on other stories
- **User Story 2 (P2)**: Can start after Foundational - Enhances US1 with real-time sync (integrates with existing task operations)
- **User Story 3 (P3)**: Can start after Foundational - Adds commenting to US1 tasks (integrates with TaskDetailModal)
- **User Story 4 (P4)**: Can start after Foundational - Adds access control to US1 boards (enhances board authorization)

### Within Each User Story

- **TDD Red Phase**: Write ALL tests FIRST - they MUST fail initially
- **TDD Green Phase**: Implement code to make tests pass
  - Models/Services before API endpoints
  - API endpoints before frontend components
  - Core functionality before real-time integration
- **TDD Refactor Phase**: Clean up code while keeping tests green
- Story complete before moving to next priority

### Parallel Opportunities

- **Phase 1 Setup**: Tasks T003, T004, T008, T009, T010 can run in parallel
- **Phase 2 Foundational**: Tasks T016-T027 can run in parallel (different files, no dependencies)
- **User Story Tests**: All test tasks within a story marked [P] can run in parallel (T028-T035 for US1, T071-T074 for US2, etc.)
- **User Story Implementation**: After Phase 2 completes, all user stories CAN proceed in parallel if team capacity allows, though P1→P2→P3→P4 sequential order recommended for integration stability
- **Backend API Endpoints**: Tasks marked [P] within same story can run in parallel (e.g., T036-T039 auth endpoints)
- **Frontend Components**: Tasks marked [P] within same story can run in parallel (e.g., T057-T065 components)
- **Phase 7 Polish**: Most tasks marked [P] can run in parallel as they are independent improvements

---

## Parallel Example: User Story 1 Setup

```bash
# Launch all User Story 1 tests together (TDD red phase):
Task T028: "Write backend integration test for POST /api/v1/auth/register"
Task T029: "Write backend integration test for POST /api/v1/auth/login"
Task T030: "Write backend integration test for POST /api/v1/boards"
Task T031: "Write backend integration test for POST /api/v1/columns"
Task T032: "Write backend integration test for POST /api/v1/tasks"
Task T033: "Write backend integration test for PUT /api/v1/tasks/:taskId/move"
Task T034: "Write frontend component test for TaskCard drag-and-drop"
Task T035: "Write E2E test for complete task creation and organization flow"

# Then launch parallel backend API endpoints:
Task T036: "Implement POST /api/v1/auth/register endpoint"
Task T037: "Implement POST /api/v1/auth/login endpoint"
Task T038: "Implement POST /api/v1/auth/logout endpoint"
Task T039: "Implement GET /api/v1/auth/me endpoint"
Task T041: "Implement POST /api/v1/boards endpoint"
Task T042: "Implement GET /api/v1/boards endpoint"
# ... etc

# Launch parallel frontend components:
Task T057: "Create LoginPage component"
Task T058: "Create RegisterPage component"
Task T059: "Create DashboardPage component"
Task T060: "Create BoardPage component"
Task T061: "Create TaskCard component"
# ... etc
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup (T001-T011)
2. Complete Phase 2: Foundational (T012-T027) - **CRITICAL BLOCKER**
3. Write User Story 1 Tests (T028-T035) - **TDD Red Phase**
4. Complete User Story 1 Implementation (T036-T070) - **TDD Green Phase**
5. **STOP and VALIDATE**: Test User Story 1 independently
6. Deploy/demo MVP if ready

**Result**: Working single-user Kanban board with authentication, board creation, task creation, and drag-and-drop

### Incremental Delivery (Recommended)

1. Complete Setup + Foundational → Foundation ready (T001-T027)
2. Add User Story 1 → Test independently → **Deploy MVP** (T028-T070)
3. Add User Story 2 → Test independently → **Deploy v1.1 with real-time** (T071-T098)
4. Add User Story 3 → Test independently → **Deploy v1.2 with comments** (T099-T119)
5. Add User Story 4 → Test independently → **Deploy v1.3 with sharing** (T120-T137)
6. Add Polish features → **Deploy v2.0 production-ready** (T138-T160)

**Each story adds value without breaking previous stories**

### Parallel Team Strategy

With multiple developers:

1. **Week 1**: Team completes Setup + Foundational together (T001-T027)
2. **Week 2-3**: Once Foundational is done:
   - Developer A: User Story 1 (T028-T070)
   - Developer B: User Story 2 (T071-T098) *can start after T027*
   - Developer C: User Story 3 (T099-T119) *can start after T027*
3. **Week 4**: 
   - Developer A: User Story 4 (T120-T137)
   - Developer B+C: Integration testing and polish
4. **Week 5**: Team on Polish & deployment prep (T138-T160)

**Stories complete and integrate independently with minimal merge conflicts**

---

## Task Count Summary

- **Phase 1 Setup**: 11 tasks
- **Phase 2 Foundational**: 16 tasks (BLOCKS all stories)
- **Phase 3 User Story 1 (P1 MVP)**: 43 tasks (8 tests + 35 implementation)
- **Phase 4 User Story 2 (P2)**: 28 tasks (4 tests + 24 implementation)
- **Phase 5 User Story 3 (P3)**: 21 tasks (4 tests + 17 implementation)
- **Phase 6 User Story 4 (P4)**: 18 tasks (4 tests + 14 implementation)
- **Phase 7 Polish**: 23 tasks
- **Total**: 160 tasks

**MVP Scope (P1 only)**: 70 tasks (Setup + Foundational + US1)
**Full Feature Set (P1-P4 + Polish)**: 160 tasks

---

## Notes

- **[P] tasks** = different files/no dependencies → can run in parallel
- **[Story] labels** = map task to specific user story for traceability
- **TDD Workflow**: Write tests FIRST (they should FAIL), then implement to make them PASS
- Each user story is independently completable and testable
- Verify tests fail before implementing (red → green → refactor)
- Commit after each task or logical group
- Stop at any checkpoint to validate story independently
- **Constitution compliance**: All tasks follow secure coding (input validation, JWT auth, bcrypt), TDD (tests first), code quality (ESLint/Prettier), accessibility (WCAG 2.1), and performance (indexing, caching) principles
