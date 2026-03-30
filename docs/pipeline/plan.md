# Implementation Plan: Palmidos AI

**Project ID:** palmidos-ai
**Author:** Cartman (CTO)
**Date:** 2026-03-30

---

## Stage 1: Database Schema & Project Scaffold

**Objective:** Set up Next.js project with Prisma schema, migrations, and core configuration

**Implements:** Foundation for R-001 through R-010

**Prerequisites:** None (first stage)

**Test strategy:** strict

### Files to Create/Modify

| File Path | Action | Purpose |
|-----------|--------|---------|
| `prisma/schema.prisma` | create | Database models (User, Integration, Calendar, Event, Task, ChatMessage) |
| `.env.example` | modify | Add required environment variables |
| `package.json` | modify | Add dependencies (prisma, @prisma/client, next-auth, openai, etc.) |
| `src/lib/db.ts` | create | Prisma client singleton |
| `src/lib/auth.ts` | create | NextAuth configuration with Google OAuth |
| `src/app/layout.tsx` | create | Root layout with providers |
| `src/app/globals.css` | create | Tailwind base styles |

### Acceptance Criteria — Stage 1

1. [ ] `prisma/schema.prisma` defines all 6 entities with correct fields, types, and relationships per architecture.md
2. [ ] `npx prisma migrate dev` runs successfully and creates migration file
3. [ ] `npx prisma generate` produces TypeScript types without errors
4. [ ] `src/lib/db.ts` exports Prisma client singleton
5. [ ] `src/lib/auth.ts` configures NextAuth with Google provider
6. [ ] `.env.example` lists all required env vars with descriptions
7. [ ] `npm install` completes without errors
8. [ ] `npm run build` compiles successfully (no type errors)

### Estimated Complexity

**Complexity:** M (2-3 hours)

---

## Stage 2: Authentication & User Setup

**Objective:** Implement Google OAuth signup/login and user dashboard shell

**Implements:** R-001

**Prerequisites:** Stage 1 complete

**Test strategy:** strict

### Files to Create/Modify

| File Path | Action | Purpose |
|-----------|--------|---------|
| `src/app/api/auth/[...nextauth]/route.ts` | create | NextAuth API route |
| `src/app/page.tsx` | create | Landing page with "Get Started" CTA |
| `src/app/(dashboard)/layout.tsx` | create | Dashboard layout with sidebar |
| `src/app/(dashboard)/page.tsx` | create | Daily briefing dashboard (placeholder) |
| `src/components/ui/button.tsx` | create | shadcn Button component |
| `src/components/ui/card.tsx` | create | shadcn Card component |
| `src/components/dashboard/Sidebar.tsx` | create | Navigation sidebar |

### Acceptance Criteria — Stage 2

1. [ ] User can click "Get Started" on landing page and complete Google OAuth flow
2. [ ] After OAuth, user is redirected to dashboard with valid session
3. [ ] Dashboard shows user avatar and name from Google account
4. [ ] Sidebar displays navigation links (Dashboard, Chat, Tasks, Settings)
5. [ ] Session persists across page reloads
6. [ ] Unauthenticated users accessing `/` (dashboard) are redirected to landing page
7. [ ] Sign out button clears session and redirects to landing page

### Estimated Complexity

**Complexity:** M (2-3 hours)

---

## Stage 3: Google Calendar Integration

**Objective:** Connect Google Calendar and sync events to database

**Implements:** R-002

**Prerequisites:** Stage 2 complete

**Test strategy:** strict

### Files to Create/Modify

| File Path | Action | Purpose |
|-----------|--------|---------|
| `src/app/api/calendars/route.ts` | create | List connected calendars |
| `src/app/api/calendars/sync/route.ts` | create | Trigger calendar sync |
| `src/app/api/events/route.ts` | create | CRUD for events |
| `src/lib/google/calendar.ts` | create | Google Calendar API wrapper |
| `src/lib/encryption.ts` | create | Token encryption utilities |
| `src/app/(dashboard)/settings/page.tsx` | create | Settings page with calendar connect |
| `src/components/dashboard/EventList.tsx` | create | Display today's events |

### Acceptance Criteria — Stage 3

1. [ ] User can connect Google Calendar from Settings page
2. [ ] OAuth tokens are encrypted before storage in database
3. [ ] Sync fetches next 30 days of events and stores in Event table
4. [ ] `GET /api/events?start=&end=` returns events within date range (raw array)
5. [ ] Dashboard displays today's events in EventList component
6. [ ] Events show title, time, and location
7. [ ] Sync can be triggered manually from settings

### Estimated Complexity

**Complexity:** L (3-4 hours)

---

## Stage 4: Task Management

**Objective:** Full CRUD for tasks with priority and due dates

**Implements:** R-005, R-008, R-010

**Prerequisites:** Stage 2 complete (can parallel with Stage 3)

**Test strategy:** strict

### Files to Create/Modify

| File Path | Action | Purpose |
|-----------|--------|---------|
| `src/app/api/tasks/route.ts` | create | List and create tasks |
| `src/app/api/tasks/[id]/route.ts` | create | Update and delete tasks |
| `src/app/(dashboard)/tasks/page.tsx` | create | Task management page |
| `src/components/tasks/TaskCard.tsx` | create | Individual task display |
| `src/components/tasks/TaskForm.tsx` | create | Add/edit task modal |
| `src/components/tasks/PriorityBadge.tsx` | create | Priority indicator |
| `src/hooks/useTasks.ts` | create | SWR hook for tasks |
| `src/lib/api/tasks.ts` | create | Typed API client for tasks |

### Acceptance Criteria — Stage 4

1. [ ] `GET /api/tasks` returns tasks filtered by status, priority, due date
2. [ ] `POST /api/tasks` creates task with title, priority, dueDate
3. [ ] `PATCH /api/tasks/[id]` updates task fields
4. [ ] `DELETE /api/tasks/[id]` removes task
5. [ ] Tasks page displays all tasks with priority badges
6. [ ] User can create task via modal form
7. [ ] User can mark task complete (checkbox)
8. [ ] Completed tasks show strikethrough and reduced opacity
9. [ ] Tasks sorted by priority (high → low) then due date

### Estimated Complexity

**Complexity:** M (2-3 hours)

---

## Stage 5: AI Assistant Chat

**Objective:** Natural language chat interface with OpenAI integration

**Implements:** R-003, R-007, R-010

**Prerequisites:** Stage 3 and 4 complete (needs calendar and task data)

**Test strategy:** strict

### Files to Create/Modify

| File Path | Action | Purpose |
|-----------|--------|---------|
| `src/app/api/chat/route.ts` | create | SSE streaming chat endpoint |
| `src/app/api/chat/history/route.ts` | create | Fetch chat history |
| `src/lib/ai/openai.ts` | create | OpenAI client setup |
| `src/lib/ai/intents.ts` | create | Intent classification |
| `src/lib/ai/actions.ts` | create | Action execution (create event, task) |
| `src/app/(dashboard)/chat/page.tsx` | create | Chat interface page |
| `src/components/chat/ChatInterface.tsx` | create | Main chat container |
| `src/components/chat/ChatBubble.tsx` | create | Message bubbles |
| `src/components/chat/MessageInput.tsx` | create | Chat input with send |
| `src/components/chat/StreamingText.tsx` | create | Typing animation |
| `src/hooks/useChat.ts` | create | SWR hook for chat |

### Acceptance Criteria — Stage 5

1. [ ] `POST /api/chat` accepts message and returns SSE stream
2. [ ] Assistant responds to "What's on my schedule today?" with today's events
3. [ ] Assistant can create task: "Remind me to call dentist Friday"
4. [ ] Assistant detects scheduling conflicts when creating events
5. [ ] Chat history persists and loads on page visit
6. [ ] User and assistant messages have distinct visual styles per design-bible
7. [ ] Streaming response shows character-by-character reveal
8. [ ] Chat interface is full-height with sticky input at bottom

### Estimated Complexity

**Complexity:** L (4-5 hours)

---

## Stage 6: Daily Briefing Dashboard

**Objective:** Unified dashboard showing calendar, tasks, and quick actions

**Implements:** R-004

**Prerequisites:** Stage 3, 4, 5 complete

**Test strategy:** pragmatic

### Files to Create/Modify

| File Path | Action | Purpose |
|-----------|--------|---------|
| `src/app/api/dashboard/briefing/route.ts` | create | Aggregated dashboard data |
| `src/app/(dashboard)/page.tsx` | modify | Replace placeholder with real dashboard |
| `src/components/dashboard/DailyBriefing.tsx` | create | Main dashboard component |
| `src/components/dashboard/QuickActions.tsx` | create | Shortcut buttons |
| `src/components/dashboard/StatCard.tsx` | create | Stats display |

### Acceptance Criteria — Stage 6

1. [ ] `GET /api/dashboard/briefing` returns {todayEvents, pendingTasks, unreadCount}
2. [ ] Dashboard displays greeting with user's name
3. [ ] Today's events shown in chronological order
4. [ ] Priority tasks (high/medium) shown with due dates
5. [ ] Quick action buttons: "Add Task", "Ask Assistant", "Schedule Meeting"
6. [ ] Stats show: meetings today, pending tasks, unread emails (placeholder)
7. [ ] Dashboard loads within 3 seconds

### Estimated Complexity

**Complexity:** M (2-3 hours)

---

## Stage 7: Gmail Integration

**Objective:** Connect Gmail and fetch email summaries

**Implements:** R-006

**Prerequisites:** Stage 3 complete (OAuth pattern established)

**Test strategy:** strict

### Files to Create/Modify

| File Path | Action | Purpose |
|-----------|--------|---------|
| `src/app/api/emails/summary/route.ts` | create | Email summary endpoint |
| `src/lib/google/gmail.ts` | create | Gmail API wrapper |
| `src/app/(dashboard)/settings/page.tsx` | modify | Add Gmail connect button |
| `src/components/dashboard/EmailPreview.tsx` | create | Email list display |

### Acceptance Criteria — Stage 7

1. [ ] User can connect Gmail from Settings (separate from Calendar)
2. [ ] `GET /api/emails/summary` returns unread email metadata (sender, subject, date)
3. [ ] Only metadata stored — no email body content in database
4. [ ] Assistant can respond to "Summarize my unread emails"
5. [ ] Email list shows sender name, subject, received time
6. [ ] Deep links open Gmail in new tab

### Estimated Complexity

**Complexity:** M (2-3 hours)

---

## Stage Summary

| Stage | Focus | Complexity | Est. Hours |
|-------|-------|------------|------------|
| 1 | Database & Scaffold | M | 2-3 |
| 2 | Authentication | M | 2-3 |
| 3 | Calendar Integration | L | 3-4 |
| 4 | Task Management | M | 2-3 |
| 5 | AI Chat | L | 4-5 |
| 6 | Dashboard | M | 2-3 |
| 7 | Gmail Integration | M | 2-3 |
| **Total** | | | **17-24 hours** |

---

## Development Notes

**Parallelization:**
- Stage 4 (Tasks) can start after Stage 2 completes (no calendar dependency)
- Stage 7 (Gmail) can start after Stage 3 completes

**Dependencies:**
- Stage 5 (AI Chat) requires both calendar and task data
- Stage 6 (Dashboard) requires all core features

**Risk Mitigation:**
- If OpenAI streaming proves complex, fallback to non-streaming responses
- If Google Calendar API rate limits hit, implement exponential backoff

**Testing Strategy:**
- Stages 1-5, 7: strict TDD — tests before implementation
- Stage 6: pragmatic — UI-focused, manual testing acceptable
