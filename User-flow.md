# Core User Flows & Sequence Diagrams

This document outlines 4 critical user flows for the SyncTask system, mapped to the project architecture.

---

## Flow 1: User Login (Authentication)

**Description:** The user exchanges credentials for a secure Session Token (JWT). This flow is critical as it involves the Auth Middleware verifying data against the Database before issuing access.

```mermaid
sequenceDiagram
    autonumber
    actor User
    participant Client as Client App (React)
    participant LB as Load Balancer
    participant API as API Server
    participant Auth as Auth Service
    participant DB as Database

    User->>Client: Enters Email/Password
    Client->>LB: POST /api/login
    LB->>API: Forward Request
    API->>DB: Query User by Email
    DB-->>API: Return User Hash
    API->>Auth: Verify Password Hash
    Auth-->>API: Result: Valid
    API->>Auth: Request Signed JWT
    Auth-->>API: Return JWT Token
    API-->>LB: 200 OK (Set-Cookie: JWT)
    LB-->>Client: 200 OK
    Client-->>User: Redirect to Dashboard
```

---

## Flow 2: Creating a New Task

**Description:** A standard transactional flow. The user creates data that must be persisted permanently. Note the Auth Check that happens before the DB write.

```mermaid
sequenceDiagram
    autonumber
    actor User
    participant Client as Client App
    participant LB as Load Balancer
    participant API as API Server
    participant Auth as Auth Middleware
    participant DB as Database

    User->>Client: Clicks "Add Task"
    Client->>LB: POST /api/tasks (Bearer Token)
    LB->>API: Forward Request
    
    note right of API: Security Check
    API->>Auth: Validate Token & Permissions
    Auth-->>API: Token Valid
    
    API->>DB: INSERT INTO tasks (title, status...)
    DB-->>API: Return New Task ID
    
    API-->>LB: 201 Created (JSON Task Data)
    LB-->>Client: 201 Created
    Client-->>User: UI updates with new Task
```

---

## Flow 3: Real-Time Card Move (The "Sync" Flow)

**Description:** This is the most unique flow. It uses WebSockets (WSS) instead of HTTP. When User A moves a card, the server pushes that update to User B instantly, bypassing the need for User B to refresh.

```mermaid
sequenceDiagram
    autonumber
    actor UserA as User A (Active)
    actor UserB as User B (Passive)
    participant ClientA as Client A
    participant ClientB as Client B
    participant LB as Load Balancer
    participant API as API Server
    participant DB as Database

    note over UserA, UserB: Both users are viewing Board #101

    UserA->>ClientA: Drags card to "Done"
    ClientA->>LB: WSS Message: {event: "MOVE_CARD", id: 5, col: "Done"}
    LB->>API: Forward WebSocket Frame
    
    par Persistence & Broadcast
        API->>DB: UPDATE tasks SET status='Done' WHERE id=5
        API->>LB: Broadcast: {event: "CARD_MOVED", id: 5}
    end
    
    LB->>ClientB: Push WSS Frame
    ClientB-->>UserB: Card animates to "Done" automatically
```

---

## Flow 4: Joining a Board (Hydration & Subscription)

**Description:** When a user opens a board, two things happen: they fetch the initial state (HTTP) and subscribe to future updates (WSS).

```mermaid
sequenceDiagram
    autonumber
    actor User
    participant Client as Client App
    participant LB as Load Balancer
    participant API as API Server
    participant DB as Database

    User->>Client: Clicks "Project Alpha" Board
    
    rect rgb(240, 248, 255)
        note right of User: Step 1: Initial Load (HTTP)
        Client->>LB: GET /api/boards/alpha/tasks
        LB->>API: Forward Request
        API->>DB: SELECT * FROM tasks WHERE board='alpha'
        DB-->>API: Return 50 tasks
        API-->>Client: JSON List of Tasks
    end

    rect rgb(255, 250, 240)
        note right of User: Step 2: Live Subscription (WebSocket)
        Client->>LB: WSS Connect / Subscribe 'board-alpha'
        LB->>API: Upgrade Connection
        API-->>Client: Confirm Subscription
        note over Client, API: Connection remains open for updates
    end
```

---

## Summary

These four flows demonstrate the core interactions in the SyncTask system:

1. **Authentication** - Secure user verification and session management
2. **Task Creation** - Standard CRUD operation with authorization
3. **Real-Time Sync** - WebSocket-based live collaboration
4. **Board Hydration** - Hybrid HTTP/WebSocket pattern for optimal UX
