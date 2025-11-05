# Data Model: Kanban Task Management Application

**Feature**: Kanban Task Management Application  
**Date**: 2025-11-05  
**Purpose**: Define database schema, entity relationships, and data validation rules

## Entity-Relationship Overview

```
User (1) ──creates──> (*) Board
User (*) ──member of──> (*) Board [via BoardMember]
Board (1) ──contains──> (*) Column
Column (1) ──contains──> (*) Task
Task (1) ──has──> (*) Comment
Task (*) ──assigned to──> (0..1) User
Comment (*) ──authored by──> (1) User
```

## Entities

### User

Represents an authenticated person using the application.

**Fields**:

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| `id` | UUID | Primary Key | Unique identifier |
| `email` | String | Unique, Not Null, Max 255 | User's email address (login credential) |
| `username` | String | Unique, Not Null, Max 50 | Display name |
| `password_hash` | String | Not Null, Max 255 | bcrypt hash of password (12 rounds) |
| `created_at` | Timestamp | Not Null, Default Now | Account creation timestamp |
| `updated_at` | Timestamp | Not Null, Default Now | Last profile update timestamp |
| `last_login_at` | Timestamp | Nullable | Last successful login timestamp |

**Relationships**:
- **Owns** many Boards (one-to-many via `owner_id` in Board)
- **Member of** many Boards (many-to-many via BoardMember junction table)
- **Assigned to** many Tasks (one-to-many via `assigned_user_id` in Task)
- **Authors** many Comments (one-to-many via `author_id` in Comment)

**Validation Rules**:
- Email must be valid format (RFC 5322)
- Email must be unique (case-insensitive)
- Username must be 3-50 characters, alphanumeric plus underscore/hyphen
- Password must be at least 8 characters (enforced pre-hash)

**Indexes**:
- Primary: `id`
- Unique: `email` (case-insensitive), `username` (case-insensitive)

---

### Board

Represents a Kanban workspace containing columns and tasks.

**Fields**:

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| `id` | UUID | Primary Key | Unique identifier |
| `name` | String | Not Null, Max 200 | Board display name |
| `description` | Text | Nullable, Max 2000 | Optional board description |
| `owner_id` | UUID | Not Null, Foreign Key → User.id | User who created the board |
| `created_at` | Timestamp | Not Null, Default Now | Board creation timestamp |
| `updated_at` | Timestamp | Not Null, Default Now | Last board update timestamp |
| `is_archived` | Boolean | Not Null, Default False | Soft delete flag |

**Relationships**:
- **Owned by** one User (many-to-one via `owner_id`)
- **Has members** many Users (many-to-many via BoardMember junction table)
- **Contains** many Columns (one-to-many via `board_id` in Column)

**Validation Rules**:
- Name must be 1-200 characters
- Description max 2000 characters if provided
- Owner must be valid User ID

**Indexes**:
- Primary: `id`
- Foreign Key: `owner_id` → User.id
- Composite: `(owner_id, is_archived)` for listing user's active boards

**Business Rules**:
- Board owner is automatically added as member with full permissions
- Deleting a board sets `is_archived = true` (soft delete)
- Archived boards are excluded from default queries

---

### BoardMember

Junction table representing user access to boards.

**Fields**:

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| `id` | UUID | Primary Key | Unique identifier |
| `board_id` | UUID | Not Null, Foreign Key → Board.id | Board being accessed |
| `user_id` | UUID | Not Null, Foreign Key → User.id | User with access |
| `role` | Enum | Not Null, Default 'member' | Access level: 'owner', 'member' |
| `invited_at` | Timestamp | Not Null, Default Now | When user was invited |
| `invited_by` | UUID | Nullable, Foreign Key → User.id | User who sent invitation |

**Relationships**:
- **Belongs to** one Board (many-to-one via `board_id`)
- **Grants access to** one User (many-to-one via `user_id`)
- **Invited by** one User (many-to-one via `invited_by`)

**Validation Rules**:
- Unique combination of (board_id, user_id) - user can only be member once
- Role must be 'owner' or 'member'

**Indexes**:
- Primary: `id`
- Unique: `(board_id, user_id)` composite unique constraint
- Index: `user_id` for finding all boards accessible to user
- Foreign Keys: `board_id` → Board.id, `user_id` → User.id, `invited_by` → User.id

**Business Rules**:
- Board owner automatically gets role='owner' BoardMember entry
- Only board owner can invite other users
- Cannot remove last owner from board

---

### Column

Represents a status category within a board (e.g., "To Do", "In Progress", "Done").

**Fields**:

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| `id` | UUID | Primary Key | Unique identifier |
| `board_id` | UUID | Not Null, Foreign Key → Board.id | Board containing this column |
| `name` | String | Not Null, Max 100 | Column display name |
| `position` | Integer | Not Null | Order within board (0-indexed) |
| `created_at` | Timestamp | Not Null, Default Now | Column creation timestamp |
| `updated_at` | Timestamp | Not Null, Default Now | Last column update timestamp |

**Relationships**:
- **Belongs to** one Board (many-to-one via `board_id`)
- **Contains** many Tasks (one-to-many via `column_id` in Task)

**Validation Rules**:
- Name must be 1-100 characters
- Position must be >= 0
- Unique (board_id, position) - no duplicate positions within same board
- Unique (board_id, name) - no duplicate column names within same board

**Indexes**:
- Primary: `id`
- Foreign Key: `board_id` → Board.id
- Composite: `(board_id, position)` for ordering columns

**Business Rules**:
- Default columns created with new board: "To Do" (pos 0), "In Progress" (pos 1), "Done" (pos 2)
- Deleting column moves all tasks to first column or prompts user
- Reordering columns updates position field for affected columns

---

### Task

Represents a work item within a column.

**Fields**:

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| `id` | UUID | Primary Key | Unique identifier |
| `board_id` | UUID | Not Null, Foreign Key → Board.id | Board containing this task |
| `column_id` | UUID | Not Null, Foreign Key → Column.id | Current column |
| `title` | String | Not Null, Max 200 | Task title |
| `description` | Text | Nullable, Max 5000 | Detailed task description |
| `priority` | Enum | Not Null, Default 'medium' | Priority: 'low', 'medium', 'high', 'urgent' |
| `due_date` | Date | Nullable | Optional due date |
| `assigned_user_id` | UUID | Nullable, Foreign Key → User.id | User assigned to task |
| `position` | Integer | Not Null | Order within column (0-indexed) |
| `created_at` | Timestamp | Not Null, Default Now | Task creation timestamp |
| `updated_at` | Timestamp | Not Null, Default Now | Last task update timestamp |
| `created_by` | UUID | Not Null, Foreign Key → User.id | User who created task |

**Relationships**:
- **Belongs to** one Board (many-to-one via `board_id`)
- **Located in** one Column (many-to-one via `column_id`)
- **Assigned to** zero or one User (many-to-one via `assigned_user_id`)
- **Created by** one User (many-to-one via `created_by`)
- **Has** many Comments (one-to-many via `task_id` in Comment)

**Validation Rules**:
- Title must be 1-200 characters
- Description max 5000 characters if provided
- Priority must be one of: 'low', 'medium', 'high', 'urgent'
- Position must be >= 0
- Unique (column_id, position) - no duplicate positions within same column
- Due date must be today or in the future (not past) when set (i.e., due_date >= current date)

**Indexes**:
- Primary: `id`
- Foreign Keys: `board_id` → Board.id, `column_id` → Column.id, `assigned_user_id` → User.id, `created_by` → User.id
- Composite: `(column_id, position)` for ordering tasks within column
- Index: `board_id` for listing all tasks in board
- Index: `assigned_user_id` for finding user's assigned tasks

**Business Rules**:
- Moving task updates both `column_id` and `position`
- Position recalculated when tasks are reordered within column
- Deleting task recalculates positions for remaining tasks
- Task history tracked via separate TaskHistory table (optional for v1)

---

### Comment

Represents a discussion entry on a task.

**Fields**:

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| `id` | UUID | Primary Key | Unique identifier |
| `task_id` | UUID | Not Null, Foreign Key → Task.id | Task being commented on |
| `author_id` | UUID | Not Null, Foreign Key → User.id | User who wrote comment |
| `content` | Text | Not Null, Max 2000 | Comment text content |
| `parent_comment_id` | UUID | Nullable, Foreign Key → Comment.id | Parent comment for threaded replies |
| `created_at` | Timestamp | Not Null, Default Now | Comment creation timestamp |
| `updated_at` | Timestamp | Not Null, Default Now | Last edit timestamp |
| `is_edited` | Boolean | Not Null, Default False | Flag indicating if comment was edited |

**Relationships**:
- **Belongs to** one Task (many-to-one via `task_id`)
- **Authored by** one User (many-to-one via `author_id`)
- **Reply to** zero or one Comment (many-to-one via `parent_comment_id`, self-referential)
- **Has** many Replies (one-to-many via `parent_comment_id`, self-referential)

**Validation Rules**:
- Content must be 1-2000 characters
- Author must have access to task's board (enforced via middleware)
- Parent comment (if set) must belong to same task

**Indexes**:
- Primary: `id`
- Foreign Keys: `task_id` → Task.id, `author_id` → User.id, `parent_comment_id` → Comment.id
- Composite: `(task_id, created_at)` for chronological comment listing
- Index: `parent_comment_id` for threaded comment queries

**Business Rules**:
- Comments can be edited by author only
- Editing sets `is_edited = true` and updates `updated_at`
- Deleting comment soft-deletes (sets content to "[deleted]") if it has replies
- Users mentioned with @username receive notification (optional for v1)

---

## Prisma Schema (Implementation)

```prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}

model User {
  id            String    @id @default(uuid())
  email         String    @unique @db.VarChar(255)
  username      String    @unique @db.VarChar(50)
  passwordHash  String    @map("password_hash") @db.VarChar(255)
  createdAt     DateTime  @default(now()) @map("created_at")
  updatedAt     DateTime  @updatedAt @map("updated_at")
  lastLoginAt   DateTime? @map("last_login_at")

  // Relationships
  ownedBoards   Board[]        @relation("BoardOwner")
  boardMembers  BoardMember[]
  assignedTasks Task[]         @relation("TaskAssignee")
  createdTasks  Task[]         @relation("TaskCreator")
  comments      Comment[]
  invitations   BoardMember[]  @relation("InvitedBy")

  @@map("users")
}

model Board {
  id          String    @id @default(uuid())
  name        String    @db.VarChar(200)
  description String?   @db.VarChar(2000)
  ownerId     String    @map("owner_id")
  createdAt   DateTime  @default(now()) @map("created_at")
  updatedAt   DateTime  @updatedAt @map("updated_at")
  isArchived  Boolean   @default(false) @map("is_archived")

  // Relationships
  owner        User          @relation("BoardOwner", fields: [ownerId], references: [id], onDelete: Cascade)
  members      BoardMember[]
  columns      Column[]
  tasks        Task[]

  @@index([ownerId, isArchived])
  @@map("boards")
}

model BoardMember {
  id         String   @id @default(uuid())
  boardId    String   @map("board_id")
  userId     String   @map("user_id")
  role       String   @default("member") @db.VarChar(20) // 'owner' or 'member'
  invitedAt  DateTime @default(now()) @map("invited_at")
  invitedBy  String?  @map("invited_by")

  // Relationships
  board      Board    @relation(fields: [boardId], references: [id], onDelete: Cascade)
  user       User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  inviter    User?    @relation("InvitedBy", fields: [invitedBy], references: [id], onDelete: SetNull)

  @@unique([boardId, userId])
  @@index([userId])
  @@map("board_members")
}

model Column {
  id        String   @id @default(uuid())
  boardId   String   @map("board_id")
  name      String   @db.VarChar(100)
  position  Int
  createdAt DateTime @default(now()) @map("created_at")
  updatedAt DateTime @updatedAt @map("updated_at")

  // Relationships
  board     Board    @relation(fields: [boardId], references: [id], onDelete: Cascade)
  tasks     Task[]

  @@unique([boardId, position])
  @@unique([boardId, name])
  @@index([boardId, position])
  @@map("columns")
}

model Task {
  id             String    @id @default(uuid())
  boardId        String    @map("board_id")
  columnId       String    @map("column_id")
  title          String    @db.VarChar(200)
  description    String?   @db.Text
  priority       String    @default("medium") @db.VarChar(20) // 'low', 'medium', 'high', 'urgent'
  dueDate        DateTime? @map("due_date") @db.Date
  assignedUserId String?   @map("assigned_user_id")
  position       Int
  createdAt      DateTime  @default(now()) @map("created_at")
  updatedAt      DateTime  @updatedAt @map("updated_at")
  createdBy      String    @map("created_by")

  // Relationships
  board          Board     @relation(fields: [boardId], references: [id], onDelete: Cascade)
  column         Column    @relation(fields: [columnId], references: [id], onDelete: Restrict)
  assignedUser   User?     @relation("TaskAssignee", fields: [assignedUserId], references: [id], onDelete: SetNull)
  creator        User      @relation("TaskCreator", fields: [createdBy], references: [id], onDelete: SetNull)
  comments       Comment[]

  @@unique([columnId, position])
  @@index([boardId])
  @@index([assignedUserId])
  @@map("tasks")
}

model Comment {
  id              String    @id @default(uuid())
  taskId          String    @map("task_id")
  authorId        String    @map("author_id")
  content         String    @db.VarChar(2000)
  parentCommentId String?   @map("parent_comment_id")
  createdAt       DateTime  @default(now()) @map("created_at")
  updatedAt       DateTime  @updatedAt @map("updated_at")
  isEdited        Boolean   @default(false) @map("is_edited")

  // Relationships
  task            Task      @relation(fields: [taskId], references: [id], onDelete: Cascade)
  author          User      @relation(fields: [authorId], references: [id], onDelete: SetNull)
  parentComment   Comment?  @relation("CommentReplies", fields: [parentCommentId], references: [id], onDelete: Restrict)
  replies         Comment[] @relation("CommentReplies")

  @@index([taskId, createdAt])
  @@index([parentCommentId])
  @@map("comments")
}
```

## State Transitions

### Task Position Management

When task moves between columns or reorders within column:

1. **Update target task**: Set new `column_id` and `position`
2. **Recalculate positions**: 
   - If moved to new column: increment positions of tasks at/after target position in new column
   - If reordered in same column: shift positions between old and new positions
3. **Broadcast event**: Notify all board members of position change

### Column Reordering

When column position changes:

1. **Update target column**: Set new `position`
2. **Shift other columns**: Increment/decrement positions of columns between old and new positions
3. **Broadcast event**: Notify all board members to reorder columns

## Data Validation Summary

| Entity | Key Validations |
|--------|----------------|
| User | Email format, unique email/username, password strength |
| Board | Name length (1-200), owner exists |
| BoardMember | Unique (board, user), valid role |
| Column | Unique (board, position), unique (board, name), name length |
| Task | Title length (1-200), valid priority enum, unique (column, position) |
| Comment | Content length (1-2000), author has board access |

## Performance Considerations

### Indexing Strategy (Implemented)

- All foreign keys indexed automatically by PostgreSQL
- Composite indexes for common queries: `(board_id, position)`, `(task_id, created_at)`
- Unique constraints double as indexes for constraint checking

### Query Optimization

- Use Prisma `include` for eager loading related data (avoid N+1 queries)
- Paginate task lists for boards with 100+ tasks
- Use database-level cascading deletes (onDelete: Cascade) for referential integrity

### Scalability Notes

- Position recalculation can be optimized with fractional indexing (future enhancement)
- Consider read replicas for boards with high view traffic
- Archive old boards/tasks to separate tables after 1 year (soft delete approach enables this)

## Next Steps

1. ✅ Data model complete with Prisma schema
2. → Generate API contracts based on entities and relationships
3. → Create database migration from Prisma schema
4. → Implement Prisma middleware for authorization checks
