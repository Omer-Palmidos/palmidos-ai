# PRD: Palmidos AI

**Project ID:** palmidos-ai
**Author:** Kyle (Product Manager)
**Date:** 2026-03-30

---

## Problem Statement

Knowledge workers and busy professionals struggle to manage fragmented productivity tools — calendars, email, task lists, and messaging apps operate in silos. This context-switching costs the average worker 23 minutes per interruption (UC Irvine study) and leads to missed deadlines, forgotten commitments, and communication gaps. Existing solutions like basic calendar apps or standalone task managers don't provide intelligent coordination across these channels. Users need a unified command center that proactively manages their schedule, communications, and tasks through natural conversation with an AI assistant.

## Target Users

**Primary: The Overcommitted Professional**
- Mid-level to senior professionals (managers, consultants, freelancers)
- Uses 5+ productivity apps daily (calendar, email, Slack, task manager, notes)
- Primary goal: Reduce administrative overhead and never miss important commitments
- Pain point: Spending 2-3 hours daily on scheduling, email triage, and task organization

**Secondary: The Small Business Owner**
- Solopreneurs or 2-10 person business operators
- Wears multiple hats — sales, support, operations
- Primary goal: Stay on top of client communications and appointments without hiring an assistant
- Pain point: Missing follow-ups and opportunities due to disorganized communication channels

## Product Overview

Palmidos AI is a web-based AI personal assistant platform that integrates with users' existing calendar, email, and messaging services. Users interact with their assistant through a chat interface to schedule meetings, draft emails, create tasks, and get briefed on their day. The AI proactively surfaces important items, suggests optimal meeting times, and maintains context across all communication channels — acting as a single source of truth for personal productivity.

## Success Metrics

| Metric | Target | How to Measure |
|--------|--------|----------------|
| User Activation | 60% of signups connect at least 2 integrations | Track integration connections in onboarding funnel |
| Daily Active Users | 40% of users engage with assistant 3+ days/week | Measure chat sessions per user per week |
| Time Saved | Users report 5+ hours/week saved on admin tasks | In-app weekly survey + usage correlation |
| Task Completion Rate | 70% of tasks created are marked complete | Task status tracking in database |
| Retention | 50% of users return after 30 days | Cohort retention analysis |

---

## Tech Constraints

**Pre-provisioned infrastructure:**
- Hosting: Vercel (project already linked to GitHub repo)
- Database: Neon Postgres — Vercel is provisioned with **`DATABASE_URL`**
- Repo: GitHub under `tegridy-farms` org, CI/CD active from first push

**Required stack decisions:**
- Frontend framework: Next.js 14 (App Router)
- Styling: Tailwind CSS
- Auth: NextAuth.js with Google OAuth
- Key libraries: OpenAI API for assistant, Prisma ORM, date-fns

**Hard constraints:**
- Never hardcode credentials or secrets
- No external services beyond integrations listed without approval
- Database migrations must use `DATABASE_URL` env var

---

## Requirements

| ID | Priority | Description | Acceptance Criterion |
|----|----------|-------------|---------------------|
| R-001 | P0 | User can sign up and authenticate with Google OAuth | User completes OAuth flow and sees dashboard within 10 seconds |
| R-002 | P0 | User can connect Google Calendar integration | User authorizes calendar access and sees "Connected" status; events sync to database |
| R-003 | P0 | AI assistant chat interface for natural language commands | User can type "Schedule a meeting with John tomorrow at 2pm" and assistant creates calendar event |
| R-004 | P0 | Daily briefing view showing today's schedule and priorities | Dashboard displays today's meetings, pending tasks, and unread message count on load |
| R-005 | P0 | Task creation and management via chat and UI | User can create, view, complete, and delete tasks; tasks persist to database |
| R-006 | P1 | Google Gmail integration for email summaries | User can request "Summarize my unread emails" and assistant returns sender + subject list |
| R-007 | P1 | Smart meeting scheduling with conflict detection | Assistant warns "You have a conflict at 2pm — suggest 3pm instead?" when scheduling overlaps |
| R-008 | P1 | Task prioritization and due dates | User can set priority (high/medium/low) and due dates; tasks sort accordingly |
| R-009 | P2 | Weekly summary report | User receives automated weekly email with stats: meetings held, tasks completed, time saved |
| R-010 | P2 | Natural language task queries | User can ask "What do I have due this week?" and assistant lists tasks with due dates |

---

## Non-Goals (MVP Scope Boundary)

1. No mobile native app — responsive web only for MVP
2. No real-time bidirectional sync — polling-based sync every 15 minutes is acceptable
3. No Slack/Teams/Discord integrations in MVP — focus on Google ecosystem first
4. No multi-user collaboration or shared calendars — single user per account
5. No AI voice calling or phone assistant — text chat only
6. No custom AI model training — use OpenAI GPT-4 API only

---

## Implementation Notes

- Auth must be working before any integration features (R-001 before R-002)
- Calendar integration should be built before AI assistant features (R-002 before R-003) so assistant has data to work with
- Database schema for tasks and calendar events should be finalized before frontend work on dashboard
- We chose Google-first integration over Microsoft/Apple because: (1) Google Workspace has largest market share among target users, (2) Google APIs are most mature for this use case. Trade-off: excluding Microsoft 365 users in MVP.

---

## Boundaries

- Never commit `.env` files or hardcoded secrets to the repo
- Never modify the GitHub Actions / CI config unless explicitly required
- Never add npm dependencies not relevant to requirements without noting it
- Never store user email content in plain text — only metadata (sender, subject, timestamp)
- Never make more than 100 API calls per user per hour to respect rate limits

---

## Open Questions

1. Should we support multiple Google accounts per user (personal + work)?
2. What's the retention policy for synced calendar data if user disconnects integration?
3. Do we need explicit user consent for AI processing of email content for GDPR compliance?
