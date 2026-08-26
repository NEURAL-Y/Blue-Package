# Tracer BEE

A production-ready, full-stack Habit Tracker web application built with React, 
TypeScript, Express, PostgreSQL, and Docker Compose. 

Engineered specifically to solve complex timezone edge cases, enforcing strict 
server-side streak calculations, atomic write validation, and IANA local-day 
boundaries.

--------------------------------------------------------------------------------
1. ARCHITECTURAL DESIGN & LOCAL-DAY DETERMINISM
--------------------------------------------------------------------------------

[The Timezone Paradox]
Naive streak implementations measure consecutive activity using rolling 24-hour 
windows or raw UTC timestamps. This approach fails in production:
  - Two check-ins 20 hours apart can span TWO local days (e.g., 10:00 PM Monday 
    and 6:00 PM Tuesday) -> Should increment streak.
  - Two check-ins 11 hours apart can occur within the SAME local day (e.g., 
    1:00 AM Tuesday and 12:00 PM Tuesday) -> Must reject as duplicate.

[System Execution Strategy]
1. Explicit IANA Timezone Resolution: Users are bound to a validated IANA 
   timezone string (e.g., "Asia/Kolkata", "America/New_York").
2. Write-Time Normalization: Every check-in captures the exact UTC instant, 
   translates it to the user's current IANA local date (YYYY-MM-DD), and 
   persists both the UTC timestamp and the resolved local date.
3. Database-Level Concurrency Control: Single check-in uniqueness per local day 
   is enforced at the storage engine level via a composite constraint:
   
       UNIQUE (habit_id, local_date)
       
   Race conditions and duplicate submissions are safely rejected with HTTP 409 
   Conflict responses before modifying application state.
4. Server-Side Streak Engine: Frontends are treated as untrusted presentation 
   layers. Current and longest streaks are computed strictly on the backend:
     - Current Streak: Evaluates contiguous local-day chains terminating on 
       either TODAY or YESTERDAY (granting a grace period for the current day).
     - Longest Streak: Identifies the historical maximum contiguous day sequence.
     - Out-of-Order Backfilling: Historical insertions trigger deterministic 
       re-indexing of both current and longest streak metrics.

--------------------------------------------------------------------------------
2. TECH STACK & SYSTEM DEPENDENCIES
--------------------------------------------------------------------------------

- Frontend Architecture: React 18, TypeScript, Vite, CSS Modules / Tailwind
- Backend Services:      Node.js, Express, TypeScript, Luxon (Timezone Engine)
- Persistence Layer:     PostgreSQL 15 (Acid-Compliant Storage & Indexing)
- Container Orchestration: Docker, Docker Compose (Multi-Stage Builds)

--------------------------------------------------------------------------------
3. PROJECT REPOSITORY STRUCTURE
--------------------------------------------------------------------------------

habit-tracker/
|-- backend/
|   |-- src/
|   |   |-- controllers/
|   |   |   `-- habit.controller.ts  (Request validation & response mapping)
|   |   |-- services/
|   |   |   `-- streak.service.ts   (Algorithmic streak & timezone engine)
|   |   |-- types/
|   |   |   `-- habit.types.ts      (Strict TypeScript domain interfaces)
|   |   `-- server.ts               (Express app bootstrap & middleware)
|   |-- Dockerfile                  (Node.js multi-stage build container)
|   |-- package.json
|   `-- tsconfig.json
|-- frontend/
|   |-- src/
|   |   |-- App.tsx                 (Reactive habit list & check-in UI)
|   |   `-- main.tsx
|   |-- Dockerfile                  (Nginx / Vite static hosting build)
|   |-- package.json
|   `-- vite.config.ts
|-- docker-compose.yml              (Multi-container orchestration)
`-- README.md

--------------------------------------------------------------------------------
4. LOCAL INFRASTRUCTURE BOOTSTRAP
--------------------------------------------------------------------------------

Option A: Containerized Environment (Recommended)
Orchestrate PostgreSQL, Express Backend, and React Frontend via Docker:

  docker-compose up --build -d

Endpoints:
  - React Web UI:      http://localhost:5173
  - Express REST API:  http://localhost:5000
  - Postgres Instance: localhost:5432

Option B: Host Development Setup
1. Backend Service:
     cd backend
     npm install
     npm run dev
   (Listening on http://localhost:5000)

2. Frontend Service:
     cd frontend
     npm install
     npm run dev
   (Listening on http://localhost:5173)

--------------------------------------------------------------------------------
5. REST API CONTRACT
--------------------------------------------------------------------------------

GET /api/habits
Fetches all user habits with server-evaluated current and longest streaks.

Response (200 OK):
[
  {
    "id": "habit_882f1",
    "name": "Deep Work / Code",
    "timezone": "Asia/Kolkata",
    "currentStreak": 5,
    "longestStreak": 14,
    "lastCheckInLocal": "2026-08-26"
  }
]

POST /api/checkin
Processes a new check-in attempt for a given habit ID.

Request Payload:
{
  "habitId": "habit_882f1"
}

Success Response (200 OK):
{
  "message": "Check-in successfully recorded",
  "localDay": "2026-08-26",
  "streaks": {
    "currentStreak": 6,
    "longestStreak": 14
  }
}

Conflict Error Response (409 Conflict):
{
  "error": "Duplicate entry: Habit already checked in for local day 2026-08-26"
}

--------------------------------------------------------------------------------
6. INTEGRITY TESTING & EDGE CASE VERIFICATION
--------------------------------------------------------------------------------

- Cross-Midnight Execution (20h Gap): 
  Check-In A at 23:00 (Day N) and Check-In B at 19:00 (Day N+1).
  Result: Successfully logged as 2 consecutive days. Streak increments by +1.

- Same Local Day Execution (11h Gap): 
  Check-In A at 01:00 (Day N) and Check-In B at 12:00 (Day N).
  Result: Database constraint throws duplicate violation. Express returns 409.

- Grace Period Decay:
  If user completes Day N-1 but has not yet completed Day N (Today):
  Result: Current streak remains active (reflecting Day N-1 length). 
  If Day N passes without check-in, Current Streak resets to 0 at midnight.

