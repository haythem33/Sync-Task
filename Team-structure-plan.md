# Team Structure Strategy: SyncTask

This document outlines the organizational structure required to execute the SyncTask roadmap, evolving from an agile MVP build to a robust Phase I release.

---

## 🏗️ Part 1: MVP Phase (The "Build" Structure)

**Objective:** Speed to market. Deliver the 4 Core User Flows with a single server architecture.

**Philosophy:** Vertical Slices. Avoid separating "Frontend" and "Backend" teams, as the Real-Time Sync (Flow 3) requires tight coupling between the React Client and the API/DB.

**Recommended Total Headcount:** 6-8 Developers

**Teams:** 2 Squads

### 1. Squad A: The "Experience" Squad

**Focus:** Visuals, Standard CRUD, and User Onboarding.

**Composition:** 3 Developers (2 Frontend Heavy, 1 Full Stack)

**Primary Responsibility:** The "standard" web application features.

**Ownership from User Flows:**
- **Flow 1 (Login):** Building the Login UI, redirection logic, and handling the JWT storage in the client.
- **Flow 2 (Create Task):** The "Add Task" modal, form validation, and the standard HTTP POST endpoints.
- **UI Polish:** Drag-and-drop animations (optimistic UI) before the server confirms.

### 2. Squad B: The "Sync" Squad (The Core System)

**Focus:** Data Integrity, WebSockets, and Authentication Security.

**Composition:** 3 Developers (1 Frontend, 2 Backend Heavy)

**Primary Responsibility:** The "invisible" hard parts that make the app feel real-time.

**Ownership from User Flows:**
- **Flow 3 (Real-Time Move):** Implementing the WebSocket Server, the UPDATE logic in DB, and broadcasting the event.
- **Flow 4 (Board Hydration):** Optimizing the initial SQL query (SELECT *) so the board loads fast, and handling the "Subscription" handshake.
- **Auth Middleware:** Writing the secure password hashing and JWT verification logic used by both teams.

---

## 🚀 Part 2: Phase I Transition (The "Scale" Structure)

**Objective:** Reliability and Production Readiness.

**Trigger:** You are ready to introduce the Green Components from your architecture (Redis, Workers, S3, Email).

**Change in Strategy:** The "Sync Squad" evolves into a Platform team to handle infrastructure, while the "Experience Squad" grows to focus on features.

**Recommended Total Headcount:** 8-10 Developers

**Teams:** 2 Streams (Product vs. Platform)

### 1. The Product Stream (Formerly Squad A)

**Focus:** User Facing Features & Retention.

**Composition:** 4-5 Developers (Full Stack mix)

**New Responsibilities:**
- **File Attachments (UI):** Building the UI to upload images to the new S3 integration.
- **Account Recovery:** Building the "Forgot Password" screens that trigger the Email Service.
- **Refinement:** Handling edge cases in the Board UI (e.g., "What happens if I lose internet connection?").

### 2. The Platform Engineering Stream (Formerly Squad B)

**Focus:** Stability, Scalability, and Infrastructure.

**Composition:** 3-4 Developers (Backend/DevOps Heavy)

**New Responsibilities (Mapped to Architecture):**

#### Redis Implementation:
- **Challenge:** In Phase I, you might add a second API server.
- **Task:** Implement Redis Pub/Sub so if User A connects to Server 1 and User B connects to Server 2, they still see each other's moves.

#### Async Workers:
- **Task:** Move "heavy" tasks (like sending emails or processing image uploads) off the main API and into the Worker Service.

#### Monitoring:
- **Task:** Set up Datadog/CloudWatch to alert if the Database CPU hits 80% (your Phase II trigger).

---

## 📊 Summary of Responsibilities Matrix

| Feature/Component | MVP Owner | Phase I Owner | Why the change? |
|------------------|-----------|---------------|-----------------|
| Login / Auth UI | Squad A | Product Stream | Remains a user-facing feature. |
| Auth Backend / JWT | Squad B | Platform Stream | Security becomes a platform-level concern. |
| Task CRUD (HTTP) | Squad A | Product Stream | Standard feature work. |
| Real-Time (WSS) | Squad B | Platform Stream | Needs Redis scaling in Phase I. |
| Database Schema | Squad B | Platform Stream | Needs strict migration controls as data grows. |
| File Uploads | N/A | Product Stream | New feature for Phase I. |
| Image Storage (S3) | N/A | Platform Stream | Infrastructure setup required. |
| Email Notifs | N/A | Platform Stream | Requires Worker queue setup. |

---

## 💡 Executive Summary for Stakeholders

1. **Start with two tight-knit squads.** Don't split by "Front vs Back". Split by "Simple HTTP flows" (Squad A) vs "Complex Real-time flows" (Squad B).

2. **Squad B is your risk mitigator.** They tackle the hardest technical risk (WebSockets) immediately.

3. **Pivot for Phase I.** When you need to scale, Squad B becomes your "Platform Team" to install the safety rails (Redis, Monitoring, Workers) defined in the architecture.
