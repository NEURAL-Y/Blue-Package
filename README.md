# 🐝 Tracer Bee

### Timezone-Aware Habit Tracking with a Deterministic Streak Engine

Tracer Bee is a production-oriented, full-stack habit tracking application designed around one deceptively difficult problem:

> **What does "today" actually mean?**

Instead of relying on rolling 24-hour windows or naive UTC timestamps, Tracer Bee uses **IANA timezones, local calendar days, database-level constraints, and server-authoritative streak computation** to produce deterministic and concurrency-safe habit streaks.

Because apparently, counting consecutive days needed a distributed-systems intervention.

---

## ✨ Core Features

* 🌍 **IANA Timezone Support**

  * `Asia/Kolkata`
  * `America/New_York`
  * `Europe/London`
  * and other valid IANA timezone identifiers

* 📅 **Local-Day Determinism**

  * Every check-in is mapped to the user's local calendar date.
  * Streaks are based on calendar-day continuity, not elapsed hours.

* 🔒 **Database-Enforced Uniqueness**

  * A habit can have only one check-in per local day.
  * PostgreSQL enforces:

```sql
UNIQUE (habit_id, local_date)
```

* ⚡ **Atomic Check-ins**

  * Duplicate and concurrent submissions are safely handled by the database.
  * Duplicate check-ins return `409 Conflict`.

* 🧠 **Server-Side Streak Engine**

  * The frontend never determines streak correctness.
  * Current and longest streaks are calculated by the backend.

* 🔄 **Historical Backfilling**

  * Out-of-order check-ins automatically participate in deterministic streak recalculation.

* 📈 **Current & Longest Streaks**

  * Current streak
  * Longest historical streak
  * Local-day continuity analysis

* 🐳 **Dockerized Development**

  * PostgreSQL
  * Express API
  * React frontend
  * Docker Compose orchestration

* 🔷 **TypeScript Throughout**

  * Strongly typed frontend and backend domain models.

---

# 🧠 Architecture

Tracer Bee separates presentation, business logic, and persistence.

```text
                    ┌─────────────────────┐
                    │      React UI       │
                    │   TypeScript/Vite   │
                    └──────────┬──────────┘
                               │
                               │ REST API
                               ▼
                    ┌─────────────────────┐
                    │    Express API      │
                    │    TypeScript       │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │   Streak Engine     │
                    │                     │
                    │ • Timezone resolve  │
                    │ • Local-day mapping │
                    │ • Streak analysis   │
                    │ • Validation        │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │     PostgreSQL      │
                    │                     │
                    │ • ACID transactions │
                    │ • Unique constraint │
                    │ • Persistent state  │
                    └─────────────────────┘
```

---

# 🌍 The Timezone Problem

A naive implementation might calculate streaks using timestamps:

```text
Check-in A: Monday 22:00
Check-in B: Tuesday 18:00

Difference = 20 hours
```

A rolling-24-hour implementation could incorrectly treat this as one continuous period.

Tracer Bee instead resolves both timestamps into the user's local calendar:

```text
Monday → Tuesday
```

Therefore:

```text
Day N     ✓
Day N+1   ✓

Streak = 2
```

The opposite problem is also handled.

```text
Tuesday 01:00
Tuesday 12:00
```

These timestamps are 11 hours apart, but they belong to the same local calendar day:

```text
2026-08-25
2026-08-25
```

The second check-in is therefore rejected.

---

# ⏱️ Write-Time Normalization

Every check-in begins with an exact UTC instant.

```text
UTC Timestamp
      │
      ▼
User's IANA Timezone
      │
      ▼
Local Calendar Date
      │
      ▼
YYYY-MM-DD
      │
      ▼
PostgreSQL
```

For example:

```text
UTC:
2026-08-26T04:30:00Z

Timezone:
Asia/Kolkata

Local Date:
2026-08-26
```

The system persists both the exact timestamp and the resolved local date.

This prevents later timezone calculations from changing the historical meaning of a check-in.

---

# 🔐 Database Integrity

The database is treated as an authority for critical invariants.

The central constraint is:

```sql
UNIQUE (habit_id, local_date)
```

This guarantees:

```text
Habit A + 2026-08-26 → ✓
Habit A + 2026-08-26 → ✗
Habit A + 2026-08-27 → ✓
Habit B + 2026-08-26 → ✓
```

This is particularly important during concurrent requests.

```text
             Request A
                 │
                 ▼
        ┌─────────────────┐
        │    PostgreSQL   │
        │                 │
        │ habit_id + date │
        └────────┬────────┘
                 ▲
                 │
             Request B
```

Only one transaction can successfully create the unique `(habit_id, local_date)` record.

The application converts the resulting constraint violation into:

```http
409 Conflict
```

---

# 🧮 Streak Engine

Tracer Bee does not store a streak as the primary source of truth.

Instead:

```text
Check-in History
       │
       ▼
Local Dates
       │
       ▼
Sorted Unique Days
       │
       ▼
Consecutive-Day Runs
       │
       ├───────────────┐
       ▼               ▼
Current Streak    Longest Streak
```

## Current Streak

The current streak is valid when the latest completed local day is either:

```text
TODAY
```

or:

```text
YESTERDAY
```

This provides a grace period for the current day.

For example:

```text
Today = 2026-08-26

Check-ins:
2026-08-23
2026-08-24
2026-08-25
```

Result:

```text
Current Streak = 3
```

If the user reaches:

```text
2026-08-27
```

without completing `2026-08-26`, the streak is no longer current.

No midnight cron job is required. The backend can determine validity when the streak is requested.

---

# 🏆 Longest Streak

Longest streak is calculated from historical contiguous local-day sequences.

Example:

```text
2026-08-10  ✓
2026-08-11  ✓
2026-08-12  ✓

2026-08-14  ✓
2026-08-15  ✓
```

The system identifies:

```text
Run 1 = 3 days
Run 2 = 2 days

Longest Streak = 3
```

---

# 🔄 Out-of-Order Check-ins

Historical check-ins are not required to arrive chronologically.

Suppose the database contains:

```text
2026-08-20
2026-08-22
2026-08-23
```

Then a historical check-in arrives:

```text
2026-08-21
```

The resulting sequence becomes:

```text
2026-08-20
2026-08-21
2026-08-22
2026-08-23
```

The streak engine recalculates the contiguous sequence deterministically.

This means the system does not depend on insertion order.

---

# 🧪 Edge-Case Testing

Tracer Bee is designed around timezone and concurrency edge cases rather than only testing the happy path.

### Cross-Midnight Check-in

```text
23:00 Day N
      ↓
19:00 Day N+1
```

Elapsed time:

```text
20 hours
```

Local-day difference:

```text
2 calendar dates
```

Expected:

```text
✓ Valid
✓ Two consecutive days
✓ Streak increments
```

---

### Same Local Day

```text
01:00 Day N
      ↓
12:00 Day N
```

Elapsed time:

```text
11 hours
```

Local-day difference:

```text
0 days
```

Expected:

```text
✗ Duplicate
HTTP 409 Conflict
```

---

### Concurrent Duplicate Requests

```text
Request A ───────┐
                 ├── PostgreSQL
Request B ───────┘
```

Expected:

```text
One request → 200 OK
Other request → 409 Conflict
```

The database constraint provides the final integrity guarantee.

---

### DST Boundary

The system should also be tested against:

```text
Spring Forward
Fall Back
```

because local days are not merely UTC timestamps wearing a different label.

---

# 🛠️ Tech Stack

| Layer            | Technology        |
| ---------------- | ----------------- |
| Frontend         | React 18          |
| Language         | TypeScript        |
| Build Tool       | Vite              |
| Backend          | Node.js + Express |
| Timezone Engine  | Luxon             |
| Database         | PostgreSQL 15     |
| Containerization | Docker            |
| Orchestration    | Docker Compose    |
| API              | REST              |

---

# 📁 Project Structure

```text
tracer-bee/
│
├── backend/
│   ├── src/
│   │   ├── controllers/
│   │   │   └── habit.controller.ts
│   │   │
│   │   ├── services/
│   │   │   └── streak.service.ts
│   │   │
│   │   ├── types/
│   │   │   └── habit.types.ts
│   │   │
│   │   └── server.ts
│   │
│   ├── Dockerfile
│   ├── package.json
│   └── tsconfig.json
│
├── frontend/
│   ├── src/
│   │   ├── App.tsx
│   │   └── main.tsx
│   │
│   ├── Dockerfile
│   ├── package.json
│   └── vite.config.ts
│
├── docker-compose.yml
└── README.md
```

---

# 🚀 Running with Docker

The recommended development environment uses Docker Compose.

```bash
docker-compose up --build -d
```

Services:

```text
Frontend
http://localhost:5173

Backend
http://localhost:5000

PostgreSQL
localhost:5432
```

To stop the environment:

```bash
docker-compose down
```

To rebuild from scratch:

```bash
docker-compose down
docker-compose up --build
```

---

# 💻 Running Without Docker

## Backend

```bash
cd backend
npm install
npm run dev
```

Backend:

```text
http://localhost:5000
```

## Frontend

Open another terminal:

```bash
cd frontend
npm install
npm run dev
```

Frontend:

```text
http://localhost:5173
```

---

# 🔌 REST API

## Get Habits

```http
GET /api/habits
```

Example response:

```json
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
```

---

## Create Check-in

```http
POST /api/checkin
```

Request:

```json
{
  "habitId": "habit_882f1"
}
```

Success:

```json
{
  "message": "Check-in successfully recorded",
  "localDay": "2026-08-26",
  "streaks": {
    "currentStreak": 6,
    "longestStreak": 14
  }
}
```

---

## Duplicate Check-in

When a habit has already been checked in for the current local day:

```http
409 Conflict
```

Response:

```json
{
  "error": "Duplicate entry: Habit already checked in for local day 2026-08-26"
}
```

---

# 🏗️ Design Principles

Tracer Bee follows several important architectural principles.

### 1. Server Is the Source of Truth

The frontend displays state.

It does not determine:

* local dates
* streaks
* uniqueness
* historical continuity
* database integrity

---

### 2. Database Enforces Invariants

Application checks improve user experience.

Database constraints guarantee correctness.

```text
Application validation
        +
Database constraint
        =
Defense in depth
```

---

### 3. Calendar Days, Not Rolling Hours

Streaks represent:

```text
DAY → DAY → DAY
```

not:

```text
timestamp + 24 hours
```

---

### 4. Historical Data Remains Authoritative

Streaks can be recomputed from check-in history.

This avoids making a mutable cached streak counter the only representation of truth.

---

### 5. Deterministic Results

Given the same:

```text
habit
+
timezone
+
check-in history
+
current date
```

the streak engine should produce the same result.

---

# 🔬 Reliability Model

Tracer Bee separates three different responsibilities:

```text
┌───────────────────────────────┐
│ Input Validation              │
│ "Is this request structurally │
│  valid?"                      │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│ Domain Logic                  │
│ "What local day is this and   │
│  what is the streak?"         │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│ Database Integrity            │
│ "Can this state legally exist?"│
└───────────────────────────────┘
```

This separation makes the system easier to test, reason about, and extend.

---

# 🎯 Assignment Goals

Tracer Bee demonstrates practical understanding of:

* Full-stack application architecture
* React and TypeScript
* Express REST API design
* PostgreSQL relational constraints
* ACID transactions
* Concurrent write handling
* Timezone-aware programming
* IANA timezone resolution
* Deterministic algorithms
* Backend domain logic
* Docker-based development
* Edge-case testing
* API error handling

The central engineering challenge is not the habit UI.

It is ensuring that the system answers:

> **"Was this habit completed on this user's local calendar day?"**

correctly, consistently, and safely under concurrent requests.

---

# 📌 Future Improvements

Potential extensions include:

* Authentication and multi-user support
* Habit creation/editing/deletion
* Habit-specific timezone changes with historical preservation
* PostgreSQL migrations
* Automated integration tests
* Property-based streak testing
* DST regression test suites
* OpenAPI documentation
* Rate limiting
* Structured logging
* Health-check endpoints
* Redis-backed caching for read-heavy workloads
* Observability with metrics and tracing
* CI/CD pipeline
* Production deployment

---

# 🐝 Why Tracer Bee?

A bee traces its path from one point to another.

Tracer Bee applies the same idea to habit history:

```text
Check-in
   ↓
Local Day
   ↓
Daily Chain
   ↓
Streak
   ↓
Long-Term Pattern
```

Every check-in leaves a trace.

Tracer Bee turns those traces into deterministic, timezone-correct streaks.

---

## 📄 License

Add your chosen open-source license here.

---

### Tracer Bee

**Track the day. Preserve the trace. Build the streak.**
