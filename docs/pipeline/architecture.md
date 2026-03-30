# Architecture: Palmidos AI

**Project ID:** palmidos-ai
**Author:** Stan (Software Architect)
**Date:** 2026-03-30

---

## Tech Stack

| Layer | Technology | Version | Justification |
|-------|-----------|---------|---------------|
| Language | TypeScript | 5.x | Type safety across full stack; industry standard |
| Framework | Next.js | 14 (App Router) | Server components for AI streaming; Vercel-native |
| Database | PostgreSQL (Neon) | 15+ | Serverless Postgres; already provisioned via Vercel |
| ORM | Prisma | 5.x | Type-safe queries; excellent migration tooling |
| Auth | NextAuth.js | 5 (beta) | Google OAuth built-in; session management |
| Styling | Tailwind CSS | 3.x | Utility-first; rapid UI development |
| UI Components | shadcn/ui | latest | Accessible, customizable components |
| AI | OpenAI SDK | 4.x | GPT-4 for assistant; streaming support |
| Date/Time | date-fns | 3.x | Immutable date manipulation |
| Validation | Zod | 3.x | Runtime type validation for APIs |

**Trade-offs:**
- Chose NextAuth.js v5 beta over v4 for better App Router support. Trade-off: newer, less community troubleshooting content.
- Chose Prisma over Drizzle for mature migration tooling. Trade-off: slightly larger bundle, more opinionated.

---

## Environment Variables

| Name | Environments | Consumed by | Notes |
|------|--------------|-------------|-------|
| `DATABASE_URL` | production, preview, development | Server runtime, migrations | Neon pooled connection |
| `NEXTAUTH_SECRET` | all | NextAuth | Random string for JWT encryption |
| `NEXTAUTH_URL` | all | NextAuth | Base URL for callbacks |
| `GOOGLE_CLIENT_ID` | all | NextAuth | Google OAuth app ID |
| `GOOGLE_CLIENT_SECRET` | all | NextAuth | Google OAuth secret |
| `GOOGLE_CALENDAR_API_KEY` | all | Calendar sync service | For read-only calendar operations |
| `OPENAI_API_KEY` | all | AI assistant service | GPT-4 access |
| `ENCRYPTION_KEY` | all | Integration tokens | AES-256 key for token encryption |

---

## System Architecture

### Components

| Component | Responsibility | Implements | Communicates With |
|-----------|---------------|------------|-------------------|
| Auth Service | Google OAuth, session management | R-001 | Database (User table) |
| Calendar Sync | Google Calendar API integration, polling sync | R-002, R-007 | Database (Calendar, Event tables), Google APIs |
| Task Manager | CRUD operations for tasks | R-005, R-008 | Database (Task table) |
| AI Assistant | OpenAI integration, intent parsing, response streaming | R-003, R-006, R-010 | All other services via internal APIs |
| Gmail Integration | Email metadata fetch, summaries | R-006 | Google Gmail API |
| Daily Briefing | Aggregates calendar, tasks, emails for dashboard | R-004 | Calendar, Task, Gmail services |

### Data Flow

1. **User Authentication:** Google OAuth → NextAuth creates session → User record created/updated in DB
2. **Calendar Sync:** Background job polls Google Calendar API every 15 min → Normalizes events → Stores in Event table
3. **AI Chat:** User message → Intent classification (OpenAI) → Route to appropriate service → Stream response
4. **Task Creation:** Via chat (AI parses) or UI → Validation → Database insert → Real-time UI update

### Client-Server Integration

- **Server Components:** Fetch data directly via Prisma (calendar events, tasks)
- **Client Components:** Use SWR for real-time updates (`/api/tasks`, `/api/events`)
- **AI Chat:** Server-sent events (SSE) for streaming responses
- **API Routes:** All under `/api/*` — typed wrappers in `lib/api/`

---

## Data Model

### User

**Implements:** R-001

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | String (CUID) | PK | Internal user ID |
| email | String | Unique, indexed | Google email |
| name | String | | Display name |
| image | String | nullable | Google avatar URL |
| createdAt | DateTime | | Account creation |
| updatedAt | DateTime | | Last update |

### Integration

**Implements:** R-002, R-006

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | String (CUID) | PK | Integration ID |
| userId | String | FK → User | Owner |
| provider | Enum | google_calendar, gmail | Service type |
| accessToken | String (encrypted) | | OAuth access token |
| refreshToken | String (encrypted) | nullable | OAuth refresh token |
| expiresAt | DateTime | nullable | Token expiry |
| scope | String | | Granted OAuth scopes |
| isActive | Boolean | default true | Connection status |
| lastSyncedAt | DateTime | nullable | Last successful sync |

### Calendar

**Implements:** R-002

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | String (CUID) | PK | Local calendar ID |
| userId | String | FK → User | Owner |
| googleCalendarId | String | | Google Calendar ID (primary, work, etc.) |
| name | String | | Calendar name |
| color | String | nullable | Hex color |
| isPrimary | Boolean | default false | Main calendar |

### Event

**Implements:** R-002, R-007

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | String (CUID) | PK | Local event ID |
| calendarId | String | FK → Calendar | Source calendar |
| googleEventId | String | indexed | Google event ID |
| title | String | | Event title |
| description | String | nullable | Event details |
| startTime | DateTime | indexed | Event start |
| endTime | DateTime | | Event end |
| location | String | nullable | Physical/Zoom location |
| attendees | JSON | nullable | Array of {email, name, responseStatus} |
| isAllDay | Boolean | default false | All-day event |
| recurrence | String | nullable | RRULE for recurring |
| status | Enum | confirmed, tentative, cancelled | Event status |
| createdAt | DateTime | | Record creation |
| updatedAt | DateTime | | Record update |

### Task

**Implements:** R-005, R-008, R-010

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | String (CUID) | PK | Task ID |
| userId | String | FK → User | Owner |
| title | String | | Task description |
| description | String | nullable | Detailed notes |
| priority | Enum | low, medium, high | default medium |
| status | Enum | todo, in_progress, done | default todo |
| dueDate | DateTime | nullable, indexed | Deadline |
| completedAt | DateTime | nullable | Completion timestamp |
| source | Enum | manual, chat, email | How task was created |
| createdAt | DateTime | | Record creation |
| updatedAt | DateTime | | Record update |

### ChatMessage

**Implements:** R-003

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | String (CUID) | PK | Message ID |
| userId | String | FK → User | Owner |
| role | Enum | user, assistant | Who sent it |
| content | String | | Message text |
| metadata | JSON | nullable | Intent, actions taken, etc. |
| createdAt | DateTime | | Timestamp |

### Relationships

- User 1:N Integration
- User 1:N Calendar
- User 1:N Task
- User 1:N ChatMessage
- Calendar 1:N Event

---

## API Design

### Authentication

| Method | Endpoint | Request | Response | Implements |
|--------|----------|---------|----------|------------|
| GET | `/api/auth/[...nextauth]` | - | NextAuth handlers | R-001 |

### Calendar

| Method | Endpoint | Request | Response | Implements |
|--------|----------|---------|----------|------------|
| GET | `/api/calendars` | - | [{id, name, color, isPrimary}] | R-002 |
| POST | `/api/calendars/sync` | {calendarId?} | {synced: number, errors: number} | R-002 |
| GET | `/api/events` | ?start=&end= | [{id, title, startTime, endTime, ...}] | R-004 |
| POST | `/api/events` | {title, startTime, endTime, ...} | {id, ...} | R-003 |
| PATCH | `/api/events/:id` | partial Event | updated Event | R-003 |
| DELETE | `/api/events/:id` | - | {success: true} | R-003 |

**Consumer Contract:** GET `/api/events` returns raw array. Consumers MUST parse as array.

### Tasks

| Method | Endpoint | Request | Response | Implements |
|--------|----------|---------|----------|------------|
| GET | `/api/tasks` | ?status=&priority=&dueBefore= | [{id, title, priority, status, dueDate}] | R-005, R-010 |
| POST | `/api/tasks` | {title, priority?, dueDate?} | created Task | R-005 |
| PATCH | `/api/tasks/:id` | partial Task | updated Task | R-005, R-008 |
| DELETE | `/api/tasks/:id` | - | {success: true} | R-005 |

### AI Assistant

| Method | Endpoint | Request | Response | Implements |
|--------|----------|---------|----------|------------|
| POST | `/api/chat` | {message, conversationId?} | SSE stream | R-003 |
| GET | `/api/chat/history` | ?limit= | [{role, content, createdAt}] | R-003 |

**Consumer Contract:** POST `/api/chat` returns Server-Sent Events stream. Client must use EventSource.

### Gmail

| Method | Endpoint | Request | Response | Implements |
|--------|----------|---------|----------|------------|
| GET | `/api/emails/summary` | ?unreadOnly=true&limit= | [{id, from, subject, snippet, date}] | R-006 |

### Dashboard

| Method | Endpoint | Request | Response | Implements |
|--------|----------|---------|----------|------------|
| GET | `/api/dashboard/briefing` | - | {todayEvents: [], pendingTasks: [], unreadCount: number} | R-004 |

---

## Directory Structure

```
palmidos-ai/
  .github/
    workflows/
      ci.yml
  prisma/
    schema.prisma
    migrations/
  src/
    app/
      (dashboard)/
        page.tsx              # Daily briefing dashboard
        layout.tsx            # Dashboard shell with nav
        chat/
          page.tsx            # AI assistant chat interface
        tasks/
          page.tsx            # Task management
        settings/
          page.tsx            # Integrations, account settings
      api/
        auth/[...nextauth]/
          route.ts
        calendars/
          route.ts
          sync/
            route.ts
        events/
          route.ts
        tasks/
          route.ts
        chat/
          route.ts
        emails/
          route.ts
        dashboard/
          briefing/
            route.ts
      page.tsx                # Landing page
      layout.tsx              # Root layout
      globals.css
    components/
      dashboard/
        DailyBriefing.tsx
        EventList.tsx
        TaskList.tsx
        QuickActions.tsx
      chat/
        ChatInterface.tsx
        MessageBubble.tsx
        StreamingText.tsx
      tasks/
        TaskCard.tsx
        TaskForm.tsx
        PriorityBadge.tsx
      ui/                     # shadcn components
    lib/
      api/
        calendars.ts
        events.ts
        tasks.ts
        chat.ts
        emails.ts
      ai/
        openai.ts           # OpenAI client setup
        intents.ts          # Intent classification
        actions.ts          # Action execution (create event, etc.)
      auth.ts               # NextAuth config
      db.ts                 # Prisma client singleton
      encryption.ts         # Token encryption utilities
      google/
        calendar.ts         # Google Calendar API wrapper
        gmail.ts            # Gmail API wrapper
    hooks/
      useTasks.ts
      useEvents.ts
      useChat.ts
    types/
      index.ts
  .env.example
  next.config.js
  tailwind.config.ts
  tsconfig.json
```

---

## Local Development, CI, and Post-Deploy

| Topic | Convention |
|-------|------------|
| **Local env** | Copy `.env.example` → `.env.local`; get `DATABASE_URL` from project `.database-url` file |
| **Migrations** | `npx prisma migrate dev` (local), `npx prisma migrate deploy` (production) |
| **CI** | Use `/home/openclaw/.openclaw/company/templates/github-actions-ci-nextjs.yml` |
| **Post-deploy smoke** | `GET /api/dashboard/briefing` (auth required), `GET /api/calendars` (auth required), Landing page load |
| **Security scan** | Run `bash /home/openclaw/.openclaw/scripts/agentshield-scan.sh` before merge |

---

## Security Considerations

1. **OAuth tokens encrypted at rest:** Integration.accessToken and refreshToken use AES-256-GCM encryption with ENCRYPTION_KEY
2. **Row-level security:** All queries filtered by userId from session; no cross-user data access
3. **Input validation:** All API inputs validated with Zod schemas
4. **CSRF protection:** NextAuth handles CSRF tokens automatically
5. **Rate limiting:** OpenAI API calls limited to 100/hour per user; Google API calls respect quota
6. **No email content stored:** Gmail integration only stores metadata (sender, subject, date), not body content

---

## Performance Considerations

1. **Database indexing:** Event.startTime, Task.dueDate, Event.googleEventId indexed for fast queries
2. **Calendar sync:** Incremental sync using sync tokens; only fetch changed events
3. **AI streaming:** Server-sent events for chat responses; no waiting for full generation
4. **SWR caching:** Client-side stale-while-revalidate for tasks/events; 60s revalidation
5. **Image optimization:** Next.js Image component for avatars and icons

---

## Third-Party Dependencies

| Package | Version | Purpose | Justification |
|---------|---------|---------|---------------|
| next | 14.x | Framework | App Router, server components |
| next-auth | 5.x | Authentication | Google OAuth, sessions |
| @prisma/client | 5.x | Database ORM | Type-safe queries |
| prisma | 5.x | Migrations, CLI | Schema management |
| openai | 4.x | AI integration | GPT-4 API |
| tailwindcss | 3.x | Styling | Utility-first CSS |
| @radix-ui/* | latest | UI primitives | Accessible components |
| swr | 2.x | Data fetching | Client-side caching |
| date-fns | 3.x | Date manipulation | Immutable, tree-shakable |
| zod | 3.x | Validation | Runtime type safety |

---

## External Services

| Service | Purpose | Auth Model | Implements |
|---------|---------|------------|------------|
| Google OAuth | User authentication | OAuth 2.0 (NextAuth) | R-001 |
| Google Calendar API | Calendar read/write | OAuth 2.0 (user tokens) | R-002, R-003, R-007 |
| Gmail API | Email metadata read | OAuth 2.0 (user tokens) | R-006 |
| OpenAI API | AI assistant responses | API key (server-side) | R-003, R-006, R-010 |
