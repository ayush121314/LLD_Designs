# Task Management System

A scalable backend system for managing tasks, assignments, comments, and workflows — inspired by tools like Jira, Linear, GitHub Issues, and Asana.

---

## Table of Contents

- [Overview](#overview)
- [Functional Requirements](#functional-requirements)
- [System Architecture](#system-architecture)
- [Architecture Layers](#architecture-layers)
  - [Controller Layer](#1-controller-layer)
  - [Service Layer](#2-service-layer)
  - [Repository Layer](#3-repository-layer)
- [Domain Entities](#domain-entities)
- [Task Lifecycle Management](#task-lifecycle-management)
- [Event-Driven Architecture](#event-driven-architecture)
- [Observer System](#observer-system)
- [Design Patterns](#design-patterns)
- [Complete Flow Examples](#complete-flow-examples)
- [Future Improvements](#future-improvements)

---

## Overview

This system provides a robust backend for task lifecycle management. It supports task creation, status transitions, user assignments, comments, audit logging, and real-time notifications — all built on clean architectural principles.

**Core principles:**
- Modular, layered architecture
- Clear separation of concerns
- Event-driven extensibility via Observer Pattern
- State-based workflow management via State Pattern
- Loose coupling between components

---

## Functional Requirements

### Task Management
- Create tasks with title, description, priority, and creator info
- Update task title, description, and priority
- Retrieve task by ID
- Support parent-child task relationships (subtasks)

### Task Lifecycle
- Tasks have a status: `TODO`, `IN_PROGRESS`, `BLOCKED`, `DONE`
- Status transitions enforced via State Pattern
- Valid transitions:
  - `TODO → IN_PROGRESS`
  - `IN_PROGRESS → DONE`
  - `IN_PROGRESS → BLOCKED`
  - `BLOCKED → IN_PROGRESS`

### Task Assignment
- Assign one or more users to a task
- Assignment events are recorded and trigger notifications
- Stored in the `TaskAssignment` table

### Comment System
- Users can add comments to tasks
- Comment creation triggers domain events for observers

### Activity Tracking (Audit Logs)
All actions are logged with: `taskId`, `action`, `oldValue`, `newValue`, `actor`, `timestamp`

Tracked events:
- Task creation
- Status changes
- Comments added
- Task updates
- User assignments

### Event System
Domain events published on significant actions:

| Event | Trigger |
|---|---|
| `TaskCreated` | New task created |
| `TaskUpdated` | Task fields modified |
| `StatusChanged` | Task status transitioned |
| `CommentAdded` | Comment posted |
| `AssigneeAdded` | User assigned to task |

### Notification System
Observers react to events and send notifications:
- `EmailService` — email notifications
- `SlackService` — Slack notifications
- `ActivityHistoryService` — audit log recording

### Permission Checks
- Validate user permissions before actions (e.g., `canUserComment(userId, taskId)`)
- Validate task existence
- Validate state transitions

---

## System Architecture

The backend follows a layered architecture:

```
Client
  ↓
Controllers       ← HTTP request handling
  ↓
Services          ← Business logic
  ↓
Repositories      ← Data persistence
  ↓
Database
```

Cross-cutting systems:
```
State Machine Layer       ← Task lifecycle transitions
Event Publishing System   ← Domain event dispatch
Observer Services         ← Side-effect handlers
```

---

## Architecture Layers

### 1. Controller Layer

Controllers expose REST APIs and delegate to services. They contain **no business logic**.

| Controller | Endpoint | Delegates To |
|---|---|---|
| `TaskController` | `createTask`, `updateTask`, `getTask` | `TaskService` |
| `TaskCommentController` | `addComment` | `TaskCommentService` |
| `TaskAssignmentController` | `addAssign` | `TaskAssignmentService` |

---

### 2. Service Layer

Services contain all business logic, coordinate repositories, manage workflows, and publish domain events.

#### `TaskService`
Primary service for task operations.

**Dependencies:** `TaskRepository`, `TaskStateService`, `TaskCommentService`, `TaskAssignmentService`

**Methods:**
```
createTask()
updateTask()
getTask()
canUserComment(userId, taskId)
```

---

#### `TaskStateService`
Manages task lifecycle transitions using the State Pattern.

**Dependencies:** `TaskStateFactory`, `TaskRepository`

**Methods:**
```
updateTaskStatus(taskId, newStatus)
updateTaskStatus(Task task, newStatus)
```

---

#### `TaskCommentService`
Handles comment creation and event publishing.

**Dependencies:** `CommentRepository`, `EventPublisher`

**Flow:**
```
validate permission → save comment → publish CommentAdded event
```

---

#### `TaskAssignmentService`
Handles user assignment to tasks and event publishing.

**Dependencies:** `AssignmentRepository`, `EventPublisher`

**Flow:**
```
create assignment → save assignment → publish AssigneeAdded event
```

---

### 3. Repository Layer

Repositories handle all database operations. They contain **no business logic**.

| Repository | Responsibility |
|---|---|
| `TaskRepository` | Task CRUD operations |
| `CommentRepository` | Comment persistence |
| `AssignmentRepository` | Assignment persistence |
| `ActivityHistoryRepository` | Audit log persistence |

---

## Domain Entities

### Task
```
id, title, description, createdBy, priority, status, parentTask, createdAt
```

### User
```
id, name, email
```

### Comment
```
id, taskId, userId, message
```

### TaskAssignment
```
taskId, userId, assignedAt
```
Enables many-to-many relationships between users and tasks.

### ActivityHistory
```
id, taskId, action, oldValue, newValue, performedBy, timeStamp
```

### Enums
- **TaskStatus:** `TODO`, `IN_PROGRESS`, `DONE`, `BLOCKED`
- **Priority:** `HIGH`, `MEDIUM`, `LOW`

---

## Task Lifecycle Management

The system uses the **State Pattern** for workflow management.

### `TaskState` Interface
```java
transitionTaskStatus(newStatus, Task)
```

### Concrete States

| State Class | Allowed Transitions |
|---|---|
| `TodoState` | → `IN_PROGRESS` |
| `InProgressState` | → `DONE`, `BLOCKED` |
| `BlockedState` | → `IN_PROGRESS` |
| `DoneState` | _(terminal state)_ |

### `TaskStateFactory`
Returns the correct state implementation for a given status.

```java
// Internal registry
Map<TaskStatus, TaskState> stateMap

// Method
getState(TaskStatus) → TaskState
```

Benefits: eliminates switch statements, improves extensibility, encapsulates transition rules.

---

## Event-Driven Architecture

The system uses the **Observer Pattern** to handle domain events and decouple side effects from business logic.

### `TaskEvent` Interface
All events share common fields:
```
taskId, actorUserId, timestamp
```

### Event Definitions

**`TaskCreated`**
```
taskId, title, userId
```

**`TaskUpdated`**
```
taskId, field, oldValue, newValue, actionType
```

**`StatusChanged`**
```
taskId, oldStatus, newStatus
```

**`CommentAdded`**
```
taskId, userId, commentMsg
```

**`AssigneeAdded`**
```
taskId, addedUserId
```

---

### `EventPublisher`

Central dispatcher for all domain events.

```java
// Internal registry
Map<Class, List<EventListener>> registry

// Methods
publish(event)
subscribe(eventClass, listener)
```

**Event dispatch flow:**
```
Service
  ↓
EventPublisher.publish(event)
  ↓
Finds registered listeners for event type
  ↓
listener.handle(event)  ← called for each observer
```

---

## Observer System

### `EventListener` Interface
```java
handle(event)
```

### Implementations

**`ActivityHistoryService`**
Saves audit logs for every domain event.
```
event → ActivityHistoryRepository.save()
```

**`EmailService`**
Sends email notifications on task assignment, status change, and task creation.

**`SlackService`**
Sends Slack notifications on task creation, status changes, and assignment updates.

---

## Design Patterns

| Pattern | Where Used | Benefit |
|---|---|---|
| **Layered Architecture** | Controller → Service → Repository | Separation of concerns, testability |
| **State Pattern** | `TaskState`, `TodoState`, `InProgressState`, `BlockedState`, `DoneState` | Eliminates conditional logic, encapsulates workflow rules |
| **Factory Pattern** | `TaskStateFactory` | Centralized state resolution, avoids switch statements |
| **Observer Pattern** | `EventPublisher`, `EventListener`, observer services | Decouples services from side effects, easy to extend |

---

## Complete Flow Examples

### Task Creation Flow
```
POST /tasks
  ↓
TaskController.createTask(CreateTaskRequest)
  ↓
TaskService.createTask()
  ↓
TaskRepository.save()
  ↓
EventPublisher.publish(TaskCreatedEvent)
  ↓
ActivityHistoryService.handle()
EmailService.handle()
SlackService.handle()
```

### Task Update Flow
```
PUT /tasks/{id}
  ↓
TaskController.updateTask(UpdateTaskRequest)
  ↓
TaskService.updateTask()
  ↓
TaskStateService.updateTaskStatus()   ← enforces valid transitions
  ↓
TaskRepository.save()
  ↓
EventPublisher.publish(TaskUpdatedEvent / StatusChangedEvent)
  ↓
Observers execute
```

### Comment Flow
```
POST /tasks/{id}/comments
  ↓
TaskCommentController.addComment(CommentRequest)
  ↓
TaskCommentService.addComment(taskId, userId, message)
  ↓
[permission check] canUserComment(userId, taskId)
  ↓
CommentRepository.save()
  ↓
EventPublisher.publish(CommentAddedEvent)
  ↓
ActivityHistoryService.handle()
EmailService.handle()
SlackService.handle()
```

### Assignment Flow
```
POST /tasks/{id}/assign
  ↓
TaskAssignmentController.addAssign(AssignRequest)
  ↓
TaskAssignmentService.addAssign(assigneeUserId, actorUserId, taskId)
  ↓
AssignmentRepository.save()
  ↓
EventPublisher.publish(AssigneeAddedEvent)
  ↓
ActivityHistoryService.handle()
EmailService.handle()
SlackService.handle()
```

---

## Future Improvements

### Async Event Queue
Replace synchronous event dispatch with a message broker for better scalability and reliability.
- Apache Kafka
- RabbitMQ

### Caching Layer
Introduce Redis caching for frequently accessed entities (tasks, users, assignments).

### Permission System
Expand into a dedicated `TaskPermissionService` with fine-grained access control:
```
canUserComment(userId, taskId)
canEditTask(userId, taskId)
canAssignUser(userId, taskId)
```

### Pagination
Add cursor/offset-based pagination for large dataset endpoints:
```
getTaskComments(taskId, page, size)
getTaskActivity(taskId, page, size)
getAssignedTasks(userId, page, size)
```

### Additional Observers
Plug in new listeners without modifying existing services:
- `PushNotificationService`
- `AnalyticsService`
- `WebhookService`

---

## Summary

This architecture delivers:

- **Clean layered design** — controllers, services, repositories with clear responsibilities
- **Domain-driven event system** — decoupled side effects through Observer Pattern
- **State-based workflow management** — valid-only transitions via State Pattern
- **Modular service boundaries** — each service owns a single domain concern
- **Scalable notification architecture** — new observers added without touching business logic

The system is built to grow — new states, observers, services, and integrations slot in without disrupting existing functionality.