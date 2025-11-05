# WebSocket Event Contracts

**Feature**: Kanban Task Management Application  
**Protocol**: Socket.io  
**Purpose**: Define real-time event schemas for collaborative features

## Connection & Authentication

### Client → Server: `connection`

Socket.io handshake with JWT authentication.

**Authentication**: JWT token passed via:
- Cookie: `token` (httpOnly)
- OR Query param: `?token=<jwt>` (fallback for WebSocket-only clients)

**On Success**: Socket assigned to user's rooms automatically

**On Failure**: Connection rejected with error

---

## Room Management

### Auto-Join on Board Access

When user accesses a board (via API or UI), automatically join board room.

**Room Naming**: `board:{boardId}`

**Example**: User opens board `123e4567-e89b-12d3-a456-426614174000`
- Socket joins room: `board:123e4567-e89b-12d3-a456-426614174000`

### Auto-Leave on Board Exit

When user navigates away from board, leave board room.

**Event**: Client emits `board:leave`

```typescript
{
  boardId: string // UUID
}
```

---

## Board Events

### Client → Server: `board:join`

User joins board room to receive real-time updates.

**Payload**:
```typescript
{
  boardId: string // UUID
}
```

**Server Response**: Emits `board:joined` to client

**Server Broadcast**: Emits `board:user-joined` to all other users in board room

**Authorization**: User must have access to board (member or owner)

---

### Server → Client: `board:joined`

Confirms successful join to board room.

**Payload**:
```typescript
{
  boardId: string,
  activeUsers: Array<{
    userId: string,
    username: string
  }>
}
```

---

### Server → Client (Broadcast): `board:user-joined`

Notifies other users when someone joins the board.

**Payload**:
```typescript
{
  boardId: string,
  user: {
    userId: string,
    username: string
  }
}
```

---

### Server → Client (Broadcast): `board:user-left`

Notifies other users when someone leaves the board.

**Payload**:
```typescript
{
  boardId: string,
  userId: string
}
```

---

### Client → Server: `board:update`

User updates board details (name, description).

**Payload**:
```typescript
{
  boardId: string,
  name?: string,
  description?: string
}
```

**Server Broadcast**: Emits `board:updated` to all users in board room

**Authorization**: User must be board owner

---

### Server → Client (Broadcast): `board:updated`

Board details have been updated.

**Payload**:
```typescript
{
  boardId: string,
  name: string,
  description: string | null,
  updatedBy: string, // userId
  updatedAt: string  // ISO timestamp
}
```

---

## Column Events

### Client → Server: `column:create`

User creates a new column.

**Payload**:
```typescript
{
  boardId: string,
  name: string,
  position: number
}
```

**Server Broadcast**: Emits `column:created` to all users in board room

**Authorization**: User must be board member

---

### Server → Client (Broadcast): `column:created`

New column has been created.

**Payload**:
```typescript
{
  column: {
    id: string,
    boardId: string,
    name: string,
    position: number,
    createdAt: string
  },
  createdBy: string // userId
}
```

---

### Client → Server: `column:update`

User updates column (name or position).

**Payload**:
```typescript
{
  boardId: string,
  columnId: string,
  name?: string,
  position?: number
}
```

**Server Broadcast**: Emits `column:updated` to all users in board room

---

### Server → Client (Broadcast): `column:updated`

Column has been updated.

**Payload**:
```typescript
{
  column: {
    id: string,
    boardId: string,
    name: string,
    position: number,
    updatedAt: string
  },
  updatedBy: string // userId
}
```

---

### Client → Server: `column:delete`

User deletes a column.

**Payload**:
```typescript
{
  boardId: string,
  columnId: string
}
```

**Server Broadcast**: Emits `column:deleted` to all users in board room

---

### Server → Client (Broadcast): `column:deleted`

Column has been deleted.

**Payload**:
```typescript
{
  boardId: string,
  columnId: string,
  deletedBy: string // userId
}
```

---

## Task Events

### Client → Server: `task:create`

User creates a new task.

**Payload**:
```typescript
{
  boardId: string,
  columnId: string,
  title: string,
  description?: string,
  priority?: 'low' | 'medium' | 'high' | 'urgent',
  dueDate?: string, // ISO date
  assignedUserId?: string,
  position: number
}
```

**Server Broadcast**: Emits `task:created` to all users in board room

**Authorization**: User must be board member

---

### Server → Client (Broadcast): `task:created`

New task has been created.

**Payload**:
```typescript
{
  task: {
    id: string,
    boardId: string,
    columnId: string,
    title: string,
    description: string | null,
    priority: 'low' | 'medium' | 'high' | 'urgent',
    dueDate: string | null,
    assignedUserId: string | null,
    position: number,
    createdAt: string,
    createdBy: string
  },
  createdBy: string // userId
}
```

---

### Client → Server: `task:update`

User updates task details.

**Payload**:
```typescript
{
  boardId: string,
  taskId: string,
  title?: string,
  description?: string,
  priority?: 'low' | 'medium' | 'high' | 'urgent',
  dueDate?: string | null,
  assignedUserId?: string | null
}
```

**Server Broadcast**: Emits `task:updated` to all users in board room

---

### Server → Client (Broadcast): `task:updated`

Task details have been updated.

**Payload**:
```typescript
{
  task: {
    id: string,
    boardId: string,
    columnId: string,
    title: string,
    description: string | null,
    priority: 'low' | 'medium' | 'high' | 'urgent',
    dueDate: string | null,
    assignedUserId: string | null,
    position: number,
    updatedAt: string
  },
  updatedBy: string, // userId
  changes: string[]  // fields that changed
}
```

---

### Client → Server: `task:move`

User moves task to different column or position.

**Payload**:
```typescript
{
  boardId: string,
  taskId: string,
  columnId: string,    // target column
  position: number     // target position
}
```

**Server Broadcast**: Emits `task:moved` to all users in board room

**Conflict Resolution**: Last write wins; server version is authoritative

---

### Server → Client (Broadcast): `task:moved`

Task has been moved to a different column or position.

**Payload**:
```typescript
{
  task: {
    id: string,
    boardId: string,
    columnId: string,
    position: number
  },
  movedBy: string, // userId
  previousColumnId: string,
  previousPosition: number
}
```

---

### Client → Server: `task:delete`

User deletes a task.

**Payload**:
```typescript
{
  boardId: string,
  taskId: string
}
```

**Server Broadcast**: Emits `task:deleted` to all users in board room

---

### Server → Client (Broadcast): `task:deleted`

Task has been deleted.

**Payload**:
```typescript
{
  boardId: string,
  taskId: string,
  deletedBy: string // userId
}
```

---

## Comment Events

### Client → Server: `comment:create`

User adds a comment to a task.

**Payload**:
```typescript
{
  boardId: string,
  taskId: string,
  content: string,
  parentCommentId?: string // for threaded replies
}
```

**Server Broadcast**: Emits `comment:created` to all users in board room

---

### Server → Client (Broadcast): `comment:created`

New comment has been added.

**Payload**:
```typescript
{
  comment: {
    id: string,
    taskId: string,
    authorId: string,
    content: string,
    parentCommentId: string | null,
    createdAt: string
  },
  author: {
    id: string,
    username: string
  },
  boardId: string
}
```

---

### Client → Server: `comment:update`

User edits their comment.

**Payload**:
```typescript
{
  boardId: string,
  taskId: string,
  commentId: string,
  content: string
}
```

**Server Broadcast**: Emits `comment:updated` to all users in board room

**Authorization**: User must be comment author

---

### Server → Client (Broadcast): `comment:updated`

Comment has been edited.

**Payload**:
```typescript
{
  comment: {
    id: string,
    taskId: string,
    content: string,
    updatedAt: string,
    isEdited: true
  },
  boardId: string
}
```

---

### Client → Server: `comment:delete`

User deletes their comment.

**Payload**:
```typescript
{
  boardId: string,
  taskId: string,
  commentId: string
}
```

**Server Broadcast**: Emits `comment:deleted` to all users in board room

**Authorization**: User must be comment author or board owner

---

### Server → Client (Broadcast): `comment:deleted`

Comment has been deleted.

**Payload**:
```typescript
{
  boardId: string,
  taskId: string,
  commentId: string,
  deletedBy: string // userId
}
```

---

## Presence Events

### Server → Client (Broadcast): `user:editing`

User is actively editing a task (typing indicator).

**Payload**:
```typescript
{
  boardId: string,
  taskId: string,
  user: {
    userId: string,
    username: string
  }
}
```

**Note**: Throttled to max 1 event per 2 seconds per user per task

---

### Server → Client (Broadcast): `user:stopped-editing`

User stopped editing a task.

**Payload**:
```typescript
{
  boardId: string,
  taskId: string,
  userId: string
}
```

**Note**: Auto-sent after 5 seconds of inactivity

---

## Error Events

### Server → Client: `error`

Operation failed with error.

**Payload**:
```typescript
{
  event: string,        // original event name
  message: string,      // user-friendly error message
  code: string,         // error code (e.g., 'UNAUTHORIZED', 'NOT_FOUND')
  details?: any         // additional error context
}
```

**Example**:
```typescript
{
  event: 'task:move',
  message: 'You do not have permission to move this task.',
  code: 'FORBIDDEN',
  details: { taskId: '123', boardId: '456' }
}
```

---

## Acknowledgment Pattern

For critical operations, client can request acknowledgment:

**Client → Server** (with callback):
```typescript
socket.emit('task:move', payload, (response) => {
  if (response.success) {
    console.log('Task moved successfully');
  } else {
    console.error('Error:', response.error);
  }
});
```

**Server Response**:
```typescript
{
  success: boolean,
  data?: any,      // result data if successful
  error?: string   // error message if failed
}
```

---

## TypeScript Event Map

For type-safe Socket.io implementation:

```typescript
// Server → Client events
interface ServerToClientEvents {
  'board:joined': (data: BoardJoinedEvent) => void;
  'board:user-joined': (data: UserJoinedEvent) => void;
  'board:user-left': (data: UserLeftEvent) => void;
  'board:updated': (data: BoardUpdatedEvent) => void;
  'column:created': (data: ColumnCreatedEvent) => void;
  'column:updated': (data: ColumnUpdatedEvent) => void;
  'column:deleted': (data: ColumnDeletedEvent) => void;
  'task:created': (data: TaskCreatedEvent) => void;
  'task:updated': (data: TaskUpdatedEvent) => void;
  'task:moved': (data: TaskMovedEvent) => void;
  'task:deleted': (data: TaskDeletedEvent) => void;
  'comment:created': (data: CommentCreatedEvent) => void;
  'comment:updated': (data: CommentUpdatedEvent) => void;
  'comment:deleted': (data: CommentDeletedEvent) => void;
  'user:editing': (data: UserEditingEvent) => void;
  'user:stopped-editing': (data: UserStoppedEditingEvent) => void;
  'error': (data: ErrorEvent) => void;
}

// Client → Server events
interface ClientToServerEvents {
  'board:join': (data: JoinBoardPayload) => void;
  'board:leave': (data: LeaveBoardPayload) => void;
  'board:update': (data: UpdateBoardPayload) => void;
  'column:create': (data: CreateColumnPayload) => void;
  'column:update': (data: UpdateColumnPayload) => void;
  'column:delete': (data: DeleteColumnPayload) => void;
  'task:create': (data: CreateTaskPayload) => void;
  'task:update': (data: UpdateTaskPayload) => void;
  'task:move': (data: MoveTaskPayload, callback?: (response: AckResponse) => void) => void;
  'task:delete': (data: DeleteTaskPayload) => void;
  'comment:create': (data: CreateCommentPayload) => void;
  'comment:update': (data: UpdateCommentPayload) => void;
  'comment:delete': (data: DeleteCommentPayload) => void;
}
```

---

## Rate Limiting

Events are rate-limited to prevent abuse:

| Event Type | Rate Limit |
|-----------|------------|
| `task:*` | 50 events/minute per user |
| `comment:*` | 30 events/minute per user |
| `board:*`, `column:*` | 20 events/minute per user |
| `user:editing` | 1 event/2 seconds per task per user (throttled) |

Exceeding rate limits results in `error` event with code `RATE_LIMIT_EXCEEDED`.

---

## Reconnection Strategy

Client should implement exponential backoff for reconnections:

1. Initial reconnection: Immediate
2. Subsequent reconnections: 1s, 2s, 4s, 8s, 16s, 30s (max)
3. On reconnect: Re-join all active board rooms
4. Server sends missed events since last disconnect (if < 5 minutes)

---

## Best Practices

1. **Optimistic UI Updates**: Update UI immediately, rollback on error
2. **Event Throttling**: Throttle `user:editing` events to reduce noise
3. **Room Management**: Leave rooms when navigating away to reduce unnecessary events
4. **Acknowledgments**: Use acknowledgment callbacks for critical operations (task moves, deletions)
5. **Error Handling**: Always listen for `error` events and provide user feedback
6. **Type Safety**: Use TypeScript event maps for compile-time safety
