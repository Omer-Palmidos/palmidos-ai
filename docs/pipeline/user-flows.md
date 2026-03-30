# User Flows: Palmidos AI

**Project ID:** palmidos-ai
**Author:** Kyle (Product Manager)
**Date:** 2026-03-30

---

## Flow 1: User Signup and Onboarding

**Implements:** R-001, R-002

As a new user, I want to sign up and connect my calendar so I can start using the AI assistant.

### Actor
New user visiting the landing page

### Preconditions
1. User has a Google account
2. User is on the Palmidos AI landing page

### Steps

| Step | User Action | System Response |
|------|-------------|-----------------|
| 1 | Clicks "Get Started" on landing page | Redirects to Google OAuth consent screen |
| 2 | Selects Google account and authorizes | Creates user record, redirects to onboarding wizard |
| 3 | Sees "Connect Your Calendar" step | Displays Google Calendar authorization button |
| 4 | Clicks "Connect Google Calendar" | Opens Google OAuth for calendar scope |
| 5 | Authorizes calendar access | Syncs next 30 days of events, shows success message |
| 6 | Sees "Meet Your Assistant" step | Displays AI assistant chat interface with welcome message |
| 7 | Types a test message (e.g., "What's on my schedule today?") | Assistant responds with today's calendar events |

### Success Criteria
- User completes signup in under 2 minutes
- User has at least 1 integration connected
- User successfully interacts with assistant

### Error States

| Error | Trigger | System Behavior | Recovery |
|-------|---------|-----------------|----------|
| OAuth denied | User cancels Google auth | Returns to landing page with "Sign up to continue" message | User can click Get Started again |
| Calendar permission denied | User skips calendar auth | Onboarding shows "Skip for now" with reminder to connect later | User can complete onboarding and connect later from settings |
| Sync failure | Google API error | Shows "Calendar connection failed" with retry button | User can retry or skip |

---

## Flow 2: Daily Briefing Check

**Implements:** R-004, R-005

As a returning user, I want to see my daily briefing so I know what's ahead.

### Actor
Returning logged-in user

### Preconditions
1. User is authenticated
2. User has connected at least one calendar

### Steps

| Step | User Action | System Response |
|------|-------------|-----------------|
| 1 | Opens app or refreshes dashboard | Loads daily briefing view automatically |
| 2 | Views "Today's Schedule" section | Displays list of today's meetings with times and attendees |
| 3 | Views "Priority Tasks" section | Displays incomplete tasks sorted by priority and due date |
| 4 | Views "Quick Actions" section | Shows buttons: "Add Task", "Schedule Meeting", "Ask Assistant" |
| 5 | Clicks "Add Task" | Opens task creation modal |
| 6 | Enters task name and clicks Save | Creates task, adds to Priority Tasks list |

### Success Criteria
- Dashboard loads within 3 seconds
- All calendar events for today are displayed
- User can create a task in under 30 seconds

### Error States

| Error | Trigger | System Behavior | Recovery |
|-------|---------|-----------------|----------|
| Calendar sync stale | Last sync > 15 min ago | Shows "Calendar syncing..." spinner with last updated timestamp | Auto-retries sync in background |
| No calendar connected | User skipped onboarding | Shows "Connect Calendar" CTA in Today's Schedule section | Click opens settings to connect |
| Task creation fails | Database error | Shows error toast "Couldn't save task — try again" | User can retry |

---

## Flow 3: AI Assistant Scheduling

**Implements:** R-003, R-007

As a user, I want to schedule meetings by chatting with my AI assistant.

### Actor
Logged-in user with calendar connected

### Preconditions
1. User is on the chat interface
2. User has at least one calendar connected

### Steps

| Step | User Action | System Response |
|------|-------------|-----------------|
| 1 | Types: "Schedule a 30-min meeting with Sarah tomorrow at 2pm" | Assistant parses intent and checks calendar for conflicts |
| 2 | — | If no conflict: "I'll schedule a 30-minute meeting with Sarah tomorrow at 2:00 PM. Confirm?" |
| 3 | Replies "Yes" or clicks Confirm button | Creates calendar event, shows "Meeting scheduled" confirmation |
| 4 | — | Displays event details: title, time, attendees, calendar link |

### Alternative Path: Conflict Detected

| Step | User Action | System Response |
|------|-------------|-----------------|
| 2a | — | If conflict exists: "You have 'Team Standup' at 2:00 PM. How about 3:00 PM instead?" |
| 3a | Replies "3pm works" or suggests alternative | Assistant proposes new time, asks for confirmation |
| 4a | Confirms | Creates event at agreed time |

### Success Criteria
- Assistant correctly interprets scheduling intent
- Conflict detection works for overlapping events
- Event is created in user's Google Calendar within 5 seconds of confirmation

### Error States

| Error | Trigger | System Behavior | Recovery |
|-------|---------|-----------------|----------|
| Ambiguous time | "Schedule with Sarah tomorrow" (no time) | Asks "What time works for you?" | User provides time |
| Unknown attendee | "Meet with John" (not in contacts) | Asks "What's John's email address?" | User provides email |
| Calendar API failure | Google API error | Shows "Couldn't create event — calendar service unavailable" | User can retry or create manually |
| Conflicting recurring event | Weekly meeting overlaps | Suggests next available slot in the same week | User accepts or proposes alternative |

---

## Flow 4: Email Summary Request

**Implements:** R-006

As a user, I want to get a quick summary of my unread emails.

### Actor
Logged-in user with Gmail connected

### Preconditions
1. User is authenticated
2. User has connected Gmail integration

### Steps

| Step | User Action | System Response |
|------|-------------|-----------------|
| 1 | Types: "Summarize my unread emails" or clicks "Email Summary" quick action | Assistant fetches unread email metadata from Gmail API |
| 2 | — | Displays: "You have 12 unread emails. Here are the important ones:" |
| 3 | — | Lists: Sender name, subject line, received time (last 24 hours) |
| 4 | Clicks on an email item | Opens Gmail in new tab to that message (deep link) |
| 5 | Asks "Draft a reply to the one from [Sender]" | Assistant generates draft reply based on email context |

### Success Criteria
- Email summary displays within 5 seconds
- Only unread emails from last 7 days are included
- Deep links to Gmail work correctly

### Error States

| Error | Trigger | System Behavior | Recovery |
|-------|---------|-----------------|----------|
| Gmail not connected | User asks for email summary | Responds "Connect your Gmail to see email summaries" with Connect button | User connects Gmail in settings |
| No unread emails | Inbox is empty | Responds "You're all caught up! No unread emails." | — |
| Gmail API rate limit | Too many requests | Shows "Email sync paused — checking again in 15 minutes" | Auto-retries later |

---

## Flow 5: Task Management

**Implements:** R-005, R-008, R-010

As a user, I want to create, view, and manage my tasks.

### Actor
Logged-in user

### Preconditions
1. User is authenticated

### Steps

| Step | User Action | System Response |
|------|-------------|-----------------|
| 1 | Types: "Remind me to call the dentist by Friday" | Assistant creates task: "Call the dentist" with due date Friday |
| 2 | — | Confirms: "Added 'Call the dentist' due Friday, March 28th" |
| 3 | Types: "What do I have due this week?" | Lists all tasks due within 7 days, sorted by priority |
| 4 | Views task list in dashboard | Shows all tasks with checkboxes, priority badges, due dates |
| 5 | Clicks checkbox next to completed task | Marks task complete, shows strikethrough, moves to "Completed" section |
| 6 | Clicks delete icon on a task | Shows "Delete this task?" confirmation |
| 7 | Confirms deletion | Removes task from list, shows "Task deleted" toast |

### Success Criteria
- Task created via chat appears in dashboard within 2 seconds
- Task list loads all user tasks in under 2 seconds
- Completing a task updates UI immediately

### Error States

| Error | Trigger | System Behavior | Recovery |
|-------|---------|-----------------|----------|
| Ambiguous due date | "Remind me next week" (vague) | Asks "Which day next week? Monday?" | User specifies day |
| Task creation fails | Database error | Shows "Couldn't create task — try again" | User can retry |
| Task not found | Deleted elsewhere | Shows "This task no longer exists" and refreshes list | List updates to current state |
