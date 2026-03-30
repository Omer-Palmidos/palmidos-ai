# Design Bible: Palmidos AI

**Project ID:** palmidos-ai
**Author:** Wendy (UX Designer)
**Date:** 2026-03-30

---

## Design Principles

1. **Clarity Over Density** — Information is organized in clear visual hierarchies; users never feel overwhelmed by data (supports R-004 — daily briefing must be scannable in seconds)

2. **Conversational First** — The AI chat is the primary interface; all other views support or enhance the conversation (supports R-003 — natural language commands)

3. **Progressive Disclosure** — Show essential information first; details available on demand via expansion or drill-down (supports user flows — daily briefing check)

4. **Calm Confidence** — Visual design feels professional and trustworthy; no flashy animations or aggressive colors that undermine productivity focus (supports target user — overcommitted professional)

5. **Responsive Consistency** — Core flows work identically across devices; layout adapts but interaction patterns remain consistent (supports non-goal — web-only MVP with responsive design)

---

## Color Palette

| Name | Hex | Usage | Contrast Ratio (on white) |
|------|-----|-------|---------------------------|
| Primary | #0F172A | Headers, primary buttons, key actions | 15.8:1 (AAA) |
| Primary Light | #1E293B | Hover states, secondary headers | 12.1:1 (AAA) |
| Accent | #3B82F6 | Links, active states, AI assistant avatar | 3.0:1 (AA Large) |
| Accent Light | #60A5FA | Hover on accent, highlights | 2.3:1 (UI only) |
| Success | #10B981 | Task completion, success messages | 3.0:1 (AA Large) |
| Warning | #F59E0B | Medium priority, pending actions | 2.1:1 (UI only) |
| Error | #EF4444 | Errors, high priority tasks | 4.0:1 (AA) |
| Background | #FFFFFF | Main background | — |
| Background Alt | #F8FAFC | Cards, sections, chat bubbles (user) | — |
| Background Dark | #0F172A | AI assistant chat bubbles | — |
| Border | #E2E8F0 | Dividers, card borders | — |
| Text Primary | #0F172A | Body text, headings | 15.8:1 (AAA) |
| Text Secondary | #64748B | Captions, metadata, timestamps | 5.7:1 (AA) |
| Text Muted | #94A3B8 | Placeholders, disabled states | 3.1:1 (AA Large) |
| Text Inverse | #FFFFFF | Text on dark backgrounds | — |

**Priority Colors (Tasks):**
- High: #EF4444 (red)
- Medium: #F59E0B (amber)
- Low: #64748B (slate)

---

## Typography

**Font Family:** Inter (system-ui fallback)

| Element | Size | Weight | Line Height | Letter Spacing |
|---------|------|--------|-------------|----------------|
| H1 | 32px / 2rem | 700 | 1.2 | -0.02em |
| H2 | 24px / 1.5rem | 600 | 1.3 | -0.01em |
| H3 | 20px / 1.25rem | 600 | 1.4 | 0 |
| H4 | 16px / 1rem | 600 | 1.5 | 0 |
| Body Large | 18px / 1.125rem | 400 | 1.6 | 0 |
| Body | 16px / 1rem | 400 | 1.6 | 0 |
| Body Small | 14px / 0.875rem | 400 | 1.5 | 0 |
| Caption | 12px / 0.75rem | 500 | 1.4 | 0.01em |
| Button | 14px / 0.875rem | 500 | 1 | 0 |
| Label | 12px / 0.75rem | 600 | 1.4 | 0.05em |

---

## Spacing System

**Base unit:** 4px

**Scale:** 4, 8, 12, 16, 20, 24, 32, 40, 48, 64, 96

**Common patterns:**
- Card padding: 24px (6 units)
- Section gap: 32px (8 units)
- Component internal gap: 16px (4 units)
- Tight grouping: 8px (2 units)
- Page margins: 24px mobile, 48px desktop

---

## Component Library

### Button

**Variants:** primary, secondary, ghost, danger
**Sizes:** sm (32px height), md (40px height), lg (48px height)

**States:**
- Default: bg-primary, text-inverse, rounded-lg
- Hover: bg-primary-light, subtle lift shadow
- Active: scale(0.98), darker bg
- Focus: ring-2 ring-accent ring-offset-2
- Disabled: opacity-50, cursor-not-allowed
- Loading: spinner replaces icon/text

**Props:** variant, size, children, onClick, disabled, loading, iconLeft, iconRight

---

### Card

**Implements:** R-004, R-005

**Variants:** default, interactive, highlighted

**Structure:**
- Border: 1px solid border color
- Border-radius: 12px (rounded-xl)
- Shadow: none (default), sm (interactive), md (highlighted)
- Padding: 24px

**States:**
- Default: bg-background, border-border
- Hover (interactive): shadow-sm, border-accent/30
- Active: shadow-inner

---

### ChatBubble

**Implements:** R-003

**Variants:** user, assistant

**User Bubble:**
- Background: bg-background-alt
- Text: text-primary
- Border-radius: 16px 16px 4px 16px
- Max-width: 80%
- Padding: 16px

**Assistant Bubble:**
- Background: bg-background-dark
- Text: text-inverse
- Border-radius: 16px 16px 16px 4px
- Max-width: 80%
- Padding: 16px
- AI avatar: 32px circle with accent bg

**States:**
- Streaming: pulsing cursor at end of text
- Error: red border-left-4 border-error

---

### TaskCard

**Implements:** R-005, R-008

**Structure:**
- Checkbox: 20px square, rounded-md
- Title: body font, strikethrough when completed
- Priority badge: caption size, colored dot + label
- Due date: caption, text-secondary, calendar icon
- Actions: hover-revealed delete/edit icons

**States:**
- Default: bg-background, border-border
- Hover: shadow-sm, show actions
- Completed: opacity-60, strikethrough title
- High priority: left border 3px error color

---

### EventCard

**Implements:** R-004

**Structure:**
- Time column: 64px wide, start time bold, end time caption
- Content: title (body), attendees avatars (if any), location (caption)
- Color indicator: 4px left border in calendar color

**States:**
- Default: bg-background
- Current/upcoming: subtle left accent border
- Past: opacity-60

---

### Input

**Variants:** text, textarea, search

**States:**
- Default: border-border, bg-background
- Focus: border-accent, ring-2 ring-accent/20
- Error: border-error, bg-error/5
- Disabled: bg-background-alt, opacity-60

**Search Input:**
- Leading icon: Search (magnifying glass)
- Placeholder: "Ask your assistant..."
- Rounded-full

---

### PriorityBadge

**Implements:** R-008

**Variants:** high, medium, low

**Structure:**
- Dot: 8px circle in priority color
- Label: caption size, priority text
- Background: priority color at 10% opacity

---

### Avatar

**Sizes:** xs (24px), sm (32px), md (40px), lg (48px)

**Structure:**
- Circular image or initials fallback
- Border: 2px solid white
- Shadow: sm

---

### EmptyState

**Structure:**
- Icon: 48px, text-muted
- Title: h4, text-primary
- Description: body-small, text-secondary
- Action: button (optional)

---

## Page Layouts

### Landing Page

**Layout:**
- Hero: centered, max-width 800px, py-24
- Navigation: fixed top, blur backdrop
- CTA: primary button "Get Started"

**Components:**
- Navigation (logo + auth buttons)
- Hero section (headline, subhead, CTA)
- Feature grid (3 columns desktop, 1 mobile)
- Footer

**Responsive:**
- Mobile: stacked features, full-width CTA
- Desktop: side-by-side hero possible with illustration

---

### Dashboard (Daily Briefing)

**Implements:** R-004

**Layout:**
- Sidebar: 280px fixed, navigation + quick actions
- Main: flex-1, scrollable, max-width 1200px
- Grid: 2 columns (calendar | tasks) on desktop, stacked on mobile

**Content hierarchy:**
1. Greeting + date header
2. Today's schedule (EventCards)
3. Priority tasks (TaskCards)
4. Quick stats (meetings, tasks, unread)

**Components:**
- Sidebar
- DailyBriefing header
- EventList
- TaskList
- QuickActions bar

**Responsive:**
- Mobile: sidebar becomes bottom nav, single column layout
- Tablet: collapsible sidebar

---

### Chat Interface

**Implements:** R-003

**Layout:**
- Full-height container
- Message list: flex-1, scrollable, reverse order (newest bottom)
- Input area: fixed bottom, 80px height
- Max-width: 900px centered

**Content hierarchy:**
1. Welcome message (assistant)
2. Message history (ChatBubbles)
3. Suggested prompts (chips above input)
4. Input field with send button

**Components:**
- ChatContainer
- ChatBubble (user + assistant)
- MessageInput
- SuggestedPrompts
- StreamingIndicator

**Responsive:**
- Mobile: full-width, input sticks to keyboard

---

### Tasks Page

**Implements:** R-005, R-008, R-010

**Layout:**
- Header: title + "Add Task" button
- Filters: status tabs (All | To Do | Done) + priority filter
- List: vertical stack of TaskCards
- Empty state when no tasks

**Components:**
- PageHeader
- FilterTabs
- TaskCard (repeated)
- TaskForm (modal for add/edit)
- EmptyState

---

### Settings Page

**Layout:**
- Sidebar tabs: Account, Integrations, Notifications
- Content area: form sections

**Integrations Section:**
- Google Calendar: status + connect/disconnect
- Gmail: status + connect/disconnect

---

## Interaction Patterns

| Pattern | Trigger | Behavior | Duration |
|---------|---------|----------|----------|
| Page transition | Navigation | Fade in content | 200ms ease-out |
| Card hover | Mouse enter | Shadow appears, subtle lift | 150ms ease |
| Button press | Click | Scale 0.98 | 100ms ease |
| Chat message appear | New message | Fade + slide up 8px | 200ms ease-out |
| Streaming text | AI response | Character-by-character reveal | 15ms per char |
| Task complete | Checkbox click | Strikethrough animates, card fades | 300ms ease |
| Loading state | Data fetch | Skeleton pulse | 1.5s infinite |
| Toast notification | Action complete | Slide in from bottom, auto-dismiss | 300ms in, 3s hold, 300ms out |
| Modal open | Trigger click | Backdrop fade, content scale up | 200ms ease-out |
| Modal close | Close action | Reverse of open | 150ms ease-in |

---

## Responsive Breakpoints

| Name | Min Width | Max Width | Layout Changes |
|------|-----------|-----------|----------------|
| Mobile | 0px | 639px | Single column, bottom nav, full-width cards |
| Tablet | 640px | 1023px | Sidebar collapsible, 2-column grids |
| Desktop | 1024px | 1279px | Fixed sidebar, 2-column dashboard |
| Wide | 1280px+ | — | Max-width container, spacious padding |

---

## Accessibility Requirements

**Color Contrast:**
- All text meets WCAG AA (4.5:1 normal, 3:1 large)
- UI components meet 3:1 against adjacent colors

**Touch Targets:**
- Minimum 44x44px for all interactive elements
- Buttons: 40px height minimum (48px for primary actions)

**Focus Indicators:**
- Visible focus ring on all interactive elements
- Ring: 2px solid accent color, 2px offset
- Skip-to-content link for keyboard users

**ARIA Patterns:**
- Navigation: `role="navigation"`, `aria-label="Main navigation"`
- Chat: `role="log"`, `aria-live="polite"` for new messages
- Modal: `role="dialog"`, `aria-modal="true"`, focus trap
- Task list: `role="list"`, each task `role="listitem"`

**Keyboard Navigation:**
- Tab order follows visual layout
- Enter/Space activates buttons and links
- Escape closes modals and dropdowns
- Arrow keys navigate within lists and menus

**Screen Reader:**
- All images have descriptive alt text
- Icons have aria-labels when not decorative
- Live region announces AI response completion
- Form inputs have associated labels
