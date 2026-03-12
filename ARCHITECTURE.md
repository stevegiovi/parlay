# Parlay — Complete Application Workflow & Architecture

> **Purpose**: Parlay is a couples collaboration app that helps partners plan events, negotiate decisions, manage shared tasks, and communicate — all with AI-powered tone filtering to keep conversations constructive.

---

## Table of Contents

1. [Tech Stack & Architecture](#1-tech-stack--architecture)
2. [Database Schema](#2-database-schema)
3. [Signup & Authentication Flow](#3-signup--authentication-flow)
4. [Onboarding & Partner Connection](#4-onboarding--partner-connection)
5. [Event Collaboration (Parlays)](#5-event-collaboration-parlays)
6. [Task Collaboration](#6-task-collaboration)
7. [Couple Chat](#7-couple-chat)
8. [AI Systems](#8-ai-systems)
9. [Real-Time Sync](#9-real-time-sync)
10. [Navigation & Screens](#10-navigation--screens)
11. [Equity & Insights](#11-equity--insights)
12. [Demo Mode](#12-demo-mode)

---

## 1. Tech Stack & Architecture

| Layer | Technology |
|-------|-----------|
| **Frontend** | Single HTML file (~5400 lines), vanilla JS, no framework |
| **Styling** | Inline `<style>` block, CSS custom properties (design tokens) |
| **Backend** | Supabase (PostgreSQL + Auth + Realtime + Edge Functions) |
| **AI** | Claude API via Supabase Edge Function (`ai-proxy`) |
| **Hosting** | GitHub Pages (static, no build step) |
| **Fonts** | Google Fonts CDN (DM Serif Display, DM Sans) |

**Key design decisions:**
- Everything in one file — no bundler, no components, no build
- Router is a simple `register(name, renderFn)` / `go(name, params)` pattern
- All screens render by swapping `.active` class on `<div class="screen">` elements
- Global state held in module-scoped variables (`_session`, `_couple`, `_tasks`, etc.)
- Perspective-aware data: tasks/messages store absolute UIDs in DB but convert to `'me'`/`'them'` in memory

---

## 2. Database Schema

### Tables

| Table | Purpose | Key Columns |
|-------|---------|------------|
| **profiles** | User identity | `id` (UUID, = auth.uid), `name`, `email`, `phone`, `love_language` |
| **couples** | Relationship link | `id`, `spouse_1_id`, `spouse_2_id`, `invite_code` (6-char), `mediator_tone`, `equity` |
| **negotiations** | Event proposals (parlays) | `id`, `couple_id`, `topic_id`, `title`, `raw_input`, `filtered_input`, `intent_context`, `date_options` (JSON), `status`, `created_by`, `partner_raw_reply`, `partner_filtered_reply`, `partner_date_response`, `counter_dates` (JSON), `confirmed_date` (JSON), `proposal`, `resolution`, `archived` |
| **tasks** | Shared task assignments | `id`, `couple_id`, `title`, `context`, `raw_context`, `filtered_context`, `category_id`, `priority`, `due_date`, `assigned_to` (UID), `created_by` (UID), `status`, `pending_response_from` (UID), `counter_count`, `counter_message`, `filtered_counter`, `decline_reason`, `filtered_decline`, `completed_at`, `completed_by`, `deleted_at` |
| **chat_messages** | Couple chat | `id`, `couple_id`, `sender_id`, `content`, `filtered_content`, `type` (text/system/ai-insight), `created_at`, `read_at`, `delivered_at`, `reply_to`, `reactions` (JSON), `metadata`, `edited_at`, `deleted`, `pinned` |
| **event_messages** | Per-parlay discussion thread | `id`, `negotiation_id`, `sender_id`, `content`, `type`, `created_at` |
| **comm_self** | Communication preferences | `user_id`, `traits` (JSON, 5 traits rated 1-5), `setup_done` |
| **family_members** | Children/dependents | `id`, `couple_id`, `name`, `age`, `needs` |

### RPC Functions

| Function | Purpose |
|----------|---------|
| `join_couple_by_code` | SECURITY DEFINER — lets spouse_2 join a couple row they don't own yet |

### Row-Level Security

- All data tables filtered by `couple_id` matching the authenticated user's couple
- `profiles` filtered by `id = auth.uid()`
- Couple ID cached in localStorage as RLS fallback for edge cases

---

## 3. Signup & Authentication Flow

### Three auth methods:

**A. Email/Password Signup**
```
1. User enters: name, email, password (6+ chars)
2. Call supabase.auth.signUp({ email, password })
3. Create profile row: { id: user_id, name }
4. If email confirmation required → redirect to sign-in screen
5. If auto-confirmed → bootstrap() → routing
```

**B. Email/Password Sign-In**
```
1. User enters: email, password
2. Call supabase.auth.signInWithPassword({ email, password })
3. Set _session, _uid globals
4. bootstrap() → routing
```

**C. Google OAuth**
```
1. Call supabase.auth.signInWithOAuth({ provider: 'google' })
2. Redirect to Google → redirect back to app
3. onAuthStateChange fires SIGNED_IN event
4. Extract name from Google metadata
5. Create profile if new user
6. bootstrap() → routing
```

### Bootstrap Process (runs after every auth)

Bootstrap loads ALL user state in one pass:

```
1. Refresh session from Supabase
2. Find couple row (query by user ID, handle duplicates, fallbacks)
3. Determine user position (am I spouse_1 or spouse_2?)
4. Cache couple_id in localStorage
5. Load my profile
6. Persist email to profile
7. Load partner's profile
8. Load shared data:
   - Parlays (negotiations table)
   - Tasks
   - Communication preferences
   - Family members
   - Chat messages
9. Subscribe to realtime updates
10. Load settings (mediator_tone, equity)
```

**Bootstrap returns:**
```javascript
{
  mode: 'supabase' | 'unauthenticated' | 'error',
  coupled: boolean,       // couple row exists
  hasPartner: boolean,    // spouse_2 filled
  onboarded: boolean,     // profile has name
  setupDone: boolean      // profile has love_language
}
```

### Post-Auth Routing Logic
```
if (pending invite code in sessionStorage):
  auto-join couple → home
else if (not coupled && no setup_done flag):
  → onboarding (ob-couple screen)
else:
  → home
```

### Global Auth State

| Variable | Type | Purpose |
|----------|------|---------|
| `_session` | object | Supabase session |
| `_uid` | string | Current user's UUID |
| `_couple` | object | Full couple row |
| `_myProfile` | object | My profile (name, email, etc.) |
| `_themProfile` | object | Partner's profile |
| `_amSpouse1` | boolean | Am I spouse_1 in the couple row? |

### Identity Helpers

| Function | Returns |
|----------|---------|
| `Auth.me()` | `{ uid, name, email }` |
| `Auth.them()` | `{ uid, name, email }` (partner) |
| `Auth.myName()` | My display name |
| `Auth.theirName()` | Partner's display name |
| `Auth.isReady()` | `!!_session && !!_couple` |
| `Auth.hasPartner()` | `!!them().uid` |

---

## 4. Onboarding & Partner Connection

### Flow for the First User (Creator)

```
ob-couple screen
  ├─ "I'm setting this up first"
  │   ├─ Auth.createCouple()
  │   │   ├─ Generate random 6-char invite code
  │   │   └─ INSERT couples row: { spouse_1_id: me, invite_code }
  │   ├─ Set localStorage flag: parlay_setup_done
  │   └─ → partner-hub screen (shows invite code, share options)
  │
  ├─ "I have an invite code"
  │   └─ → ob-join screen (enter code)
  │
  └─ "Skip for now"
      ├─ Set localStorage flag: parlay_setup_done
      └─ → home (solo mode, can connect partner later)
```

### Flow for the Second User (Joiner)

```
ob-join screen
  ├─ Enter 6-character invite code
  ├─ Auth.joinCouple(code)
  │   ├─ Validate code isn't own couple
  │   ├─ Clean up orphan solo couples (if user had one)
  │   ├─ RPC: join_couple_by_code (SECURITY DEFINER)
  │   │   └─ UPDATE couples SET spouse_2_id = me WHERE invite_code = code
  │   ├─ Re-fetch couple data
  │   ├─ Load partner's profile
  │   └─ Cache couple_id
  ├─ Toast: "Connected with your partner! 🎉"
  └─ → home
```

### Deep-Link Invite Flow

```
URL: https://parlay.app/?invite=ABC123
  ├─ On page load: save code to sessionStorage('parlay_pending_invite')
  ├─ User signs up or signs in
  ├─ Post-auth: detect pending invite
  ├─ Auto-call joinCouple(code)
  └─ → home (skips manual code entry entirely)
```

### Partner Hub Screen

The partner-hub screen (accessible from home or setup) shows:
- Connection status (connected vs. waiting)
- Invite code with copy/share buttons
- Partner's name and profile if connected
- Option to disconnect (future)

---

## 5. Event Collaboration (Parlays)

### What is a Parlay?

A **parlay** is a structured event proposal between partners. It replaces unstructured texting about plans with a negotiation workflow that includes AI tone filtering, date proposals, and fair resolution.

### Topic Categories

| ID | Icon | Label |
|----|------|-------|
| date-night | 🌹 | Date Night |
| boys-night | 🍺 | Boys' Night |
| girls-night | 🍷 | Girls' Night |
| family-visit | 👨‍👩‍👧 | Family Visit |
| trip | ✈️ | Trip / Vacation |
| hosting | 🏡 | Hosting |
| playdate | 👧 | Playdate |
| kids-activity | 🎒 | Kids' Activity |
| household | 🔧 | Household |
| other | 💬 | Other |

### Creation Flow

```
1. User taps "+ New Event Proposal" on plan screen
2. → compose-pick: Select topic from category grid
3. → compose: Fill structured intent fields
   - Topic-specific fields (location, budget, attendees, etc.)
   - Up to 4 proposed dates with time labels
   - Optional free-text note (will be tone-filtered by AI)
4. AI filters the note text before saving
5. Store.addParlay() → INSERT into negotiations table
6. Status: 'pending_partner'
7. Partner sees it in their "NEEDS YOUR RESPONSE" section
```

### Parlay Status Lifecycle

```
                    ┌──────────────┐
                    │   CREATION   │
                    │  (compose)   │
                    └──────┬───────┘
                           │
                           ▼
                 ┌──────────────────┐
                 │ pending_partner  │  Partner's turn to respond
                 └────────┬────────┘
                          │
              ┌───────────┼───────────┐
              ▼           ▼           ▼
     ┌────────────┐ ┌──────────┐ ┌──────────┐
     │  ACCEPT    │ │  REPLY   │ │ DECLINE  │
     │  a date    │ │  + note  │ │          │
     └─────┬──────┘ └────┬─────┘ └────┬─────┘
           │              │            │
           ▼              ▼            ▼
    ┌────────────┐ ┌──────────────┐ ┌──────────┐
    │  resolved  │ │partner_replied│ │ declined │
    │date_accepted│ └──────┬───────┘ └──────────┘
    └────────────┘        │
                          ▼
                  ┌───────────────┐
                  │    AI generates│
                  │    proposal    │
                  └───────┬───────┘
                          │
              ┌───────────┼──────────┐
              ▼           ▼          ▼
     ┌────────────┐ ┌──────────┐ ┌──────────────┐
     │   active   │ │in_discuss│ │counter_proposed│
     │ (proposal  │ │  ion     │ │  (new dates)  │
     │   ready)   │ └────┬─────┘ └───────┬───────┘
     └─────┬──────┘      │              │
           │              └──────┬───────┘
           ▼                     ▼
    ┌────────────┐        (back and forth
    │  resolved  │         until agreement)
    └────────────┘
```

### Status Labels by Perspective

| Status | Creator Sees | Partner Sees |
|--------|-------------|-------------|
| `pending_partner` | "Sent" 📬 | "Your turn" ⚡ |
| `partner_replied` | "Reply received" 📬 | "Awaiting proposal" ⏳ |
| `in_discussion` | "In discussion" 💬 | "In discussion" 💬 |
| `counter_proposed` | "New times" 📅 | "Counter sent" 📤 |
| `active` | "Proposal ready" ✅ | "Proposal ready" ✅ |
| `resolved` (date_accepted) | "Confirmed" ✅ | "Confirmed" ✅ |
| `declined` | "Declined" | "Declined" |

### Plan Screen Layout

The plan screen organizes parlays into smart sections:

1. **⚡ NEEDS YOUR RESPONSE** — Parlays requiring your action (filtered by status + ownership)
2. **Active** — Ongoing negotiations
3. **Upcoming** — Resolved with confirmed date (future calendar events)
4. **Resolved** — Completed negotiations (collapsed)
5. **Declined** — Rejected proposals (collapsed)

### Parlay Detail Screen

Shows full negotiation state:
- Topic icon + title + date
- Creator attribution ("From [name]" or "You proposed")
- Status badge
- Original proposal text (tone-filtered)
- Date options as selectable cards
- Partner's reply (if any)
- AI-generated fair proposal (if generated)
- Per-parlay message thread (event_messages table)
- Action buttons based on status

### Counter-Proposal Flow

```
Partner views parlay detail
  → Taps "Suggest Different Times"
  → parlay-reply screen:
    - Add up to 4 alternative dates
    - Optional note (tone-filtered)
  → Status changes to 'counter_proposed'
  → Creator sees counter-dates
  → Creator accepts one OR counters again
```

### AI Proposal Generation

When both sides have expressed preferences, the AI generates a "fair proposal":
- Considers both partners' stated needs
- Weighs date availability
- Applies mediator tone setting
- Produces structured reasoning + recommendation
- Stored in `proposal` field (JSON with reasoning)

---

## 6. Task Collaboration

### Task Data Model

| Field | Type | Purpose |
|-------|------|---------|
| `id` | string | `'task_' + Date.now()` |
| `title` | string | Task name |
| `context` / `rawContext` / `filteredContext` | string | Details (raw + AI-filtered) |
| `categoryId` | string | One of 8 categories |
| `priority` | enum | `'normal'` or `'high'` |
| `dueDate` | string | ISO date or null |
| `assignedTo` | `'me'`/`'them'` | Perspective-aware assignment |
| `createdBy` | `'me'`/`'them'` | Who created it |
| `status` | enum | See lifecycle below |
| `pendingResponseFrom` | `'me'`/`'them'`/null | Whose turn to act |
| `counterCount` | number | How many counter-proposals |
| `counterMessage` / `filteredCounter` | string | Counter-proposal text |
| `declineReason` / `filteredDecline` | string | Why declined |
| `completedAt` | timestamp | When marked done |
| `completedBy` | UID | Who completed it |

### Task Categories

| ID | Icon | Label |
|----|------|-------|
| errands | 🛒 | Errands |
| household | 🏠 | Household |
| kids | 👧 | Kids |
| fix | 🔧 | Fix & Maintenance |
| admin | 📋 | Admin |
| pets | 🐾 | Pets |
| selfcare | 💆 | Self-care |
| other | 📌 | Other |

Auto-categorization via regex matching on title keywords (e.g., "grocery" → errands, "laundry" → household).

### Task Creation Flow

```
1. User taps "+ Assign a Task" on tasks screen
2. → task-new: Select category from grid OR type custom title
   - Custom title auto-categorized via regex
3. → task-new-details: Fill in details
   - Title (editable)
   - Assign to: "Them" or "Myself" (pill selector)
   - Due date (optional date picker)
   - Priority: "Normal" or "High" (pill selector)
   - Context/details (optional textarea, tone-filtered)
4. AI filters context text
5. Store.addTask() → UPSERT into tasks table
6. Status: 'pending_response'
7. pendingResponseFrom: assignee
```

### Task Status Lifecycle

```
┌───────────────────┐
│  pending_response  │  Assignee must accept/decline
└────────┬──────────┘
         │
    ┌────┼─────┬──────────┐
    ▼    ▼     ▼          ▼
┌───────┐┌─────────┐┌──────────┐
│ACCEPT ││SUGGEST  ││ DECLINE  │
│       ││CHANGES  ││          │
└───┬───┘└────┬────┘└────┬─────┘
    │         │          │
    ▼         ▼          ▼
┌────────┐┌──────────┐┌──────────┐
│accepted││countered ││ declined │
└───┬────┘└────┬─────┘└──────────┘
    │          │
    │     (other party
    │      accepts or
    │      declines the
    │      counter)
    │          │
    ▼          ▼
┌────────┐
│  done  │  Assignee marks complete
└────────┘
```

### Task Screen Sections

1. **⚡ NEEDS YOUR ACTION** — Tasks where `pendingResponseFrom === 'me'` (red highlight)
2. **My Tasks** — Accepted tasks assigned to me (with inline "Done" button)
3. **I Assigned** — Tasks I created for partner (excluding completed)
4. **Completed** — All `status === 'done'` (collapsed by default)

### Task Detail Actions by State

| Status | Assignee Can | Creator Can |
|--------|-------------|------------|
| `pending_response` | Accept, Suggest Changes, Decline | Wait |
| `accepted` | Mark Done | View progress |
| `countered` | Wait | Accept change, Decline |
| `declined` | — | See decline reason |
| `done` | — | See completion date |

### Suggest Changes (Counter) Flow

```
1. Assignee taps "↩ Suggest Changes"
2. Form revealed for alternative suggestion
3. AI filters the counter-message
4. ToneCoach gives real-time feedback on textarea
5. Status → 'countered'
6. pendingResponseFrom flips to creator
7. Creator sees counter-proposal and can accept or decline
```

### Task Sorting Logic

1. Overdue tasks first
2. Due today next
3. By ascending due date
4. No-date tasks last

---

## 7. Couple Chat

### Architecture

- **One chat thread per couple** (not per-parlay)
- **Real-time** via Supabase Postgres Changes subscription
- **Optimistic updates** — message appears instantly, synced to DB async
- **AI-integrated** — tone filtering + contextual hints

### Message Data Model

| Field | Type | Purpose |
|-------|------|---------|
| `id` | UUID | Message ID |
| `couple_id` | UUID | Which couple |
| `sender_id` | UUID | Who sent it |
| `content` | string | Raw message text |
| `filtered_content` | string | AI tone-filtered version |
| `type` | enum | `'text'`, `'system'`, `'ai-insight'` |
| `created_at` | timestamp | When sent |
| `read_at` | timestamp | When partner read it |
| `delivered_at` | timestamp | When delivered |
| `reply_to` | UUID | If replying to another message |
| `reactions` | JSON | `[{from, emoji, ts}]` |
| `metadata` | JSON | Custom data (links, parlay refs) |
| `edited_at` | timestamp | When last edited |
| `deleted` | boolean | Soft delete flag |
| `pinned` | boolean | Pinned status |

### Send Flow

```
1. User types message in textarea
2. Draft auto-saved to localStorage (500ms debounce)
3. User taps send (↑)
4. AI tone-filters the message
5. If filter meaningfully changed text → show approval prompt:
   - "Send filtered version" vs. "Send original"
6. Create optimistic local message (temp ID)
7. Push to _messages array + render immediately
8. INSERT to Supabase async
9. Replace temp message with DB row on success
10. AI analyzes for contextual hints
```

### Receive Flow (Real-Time)

```
Supabase postgres_changes event fires
  ├─ INSERT: New message from partner
  │   ├─ Skip if already in local cache
  │   ├─ Convert DB row → in-memory format
  │   ├─ Push to _messages + re-render
  │   └─ Toast notification if not on chat screen
  ├─ UPDATE: Reaction, edit, read status change
  │   ├─ Find message by ID, replace with updated row
  │   └─ Re-render
  └─ DELETE: Message removed
      ├─ Remove from _messages
      └─ Re-render
```

### Message Status Indicators

| Icon | Meaning |
|------|---------|
| `●` | Sent (reached server) |
| `✓` | Delivered (partner's client received) |
| `✓✓` | Read (partner opened and saw it) |

### Chat UI Components

```
┌─────────────────────────────┐
│ HEADER                      │
│ [← Back] Partner Name 🟢   │
├─────────────────────────────┤
│ MESSAGE LIST                │
│                             │
│ ── Today ──                 │
│                             │
│         [Their message]  ◀  │
│  ▶  [My message]            │
│         [Their message]  ◀  │
│                             │
│ ┌─ AI Insight ────────────┐ │
│ │ 📅 Sounds like a plan!  │ │
│ │ Create a Parlay? →      │ │
│ └─────────────────────────┘ │
│                             │
├─────────────────────────────┤
│ INPUT AREA                  │
│ [+] [Message input...] [↑] │
│ [😊 Smart Replies]         │
│ [ToneCoach feedback]        │
└─────────────────────────────┘
```

### Chat Features

| Feature | Details |
|---------|---------|
| **Reply threading** | Reply to specific message; shows preview in input banner; clicking preview scrolls to original |
| **Message editing** | Own messages only; shows "(edited)" tag; tracks `edited_at` |
| **Reactions** | Multi-emoji support per message; toggle on/off; 6 quick-react buttons in context menu |
| **Pinning** | Pin/unpin messages; `pinnedMessages()` query |
| **Search** | Full-text search with result navigation (prev/next); highlights matches |
| **Message grouping** | Messages within 2 min from same sender grouped visually (adjusted border radius) |
| **Smart replies** | AI-generated quick reply suggestions shown as tappable chips |
| **Conversation starters** | Pre-written prompts when chat is empty |
| **Draft recovery** | Textarea value restored from localStorage on screen reload |
| **Export** | Download chat history |
| **Context menu** | Long-press/right-click: Reply, Copy, Pin, Edit, Delete, React |

### AI Hints in Chat

The AI analyzes each sent message and may show contextual suggestions:

| Detected Intent | Suggestion |
|-----------------|-----------|
| 📅 Scheduling | "Want to make it official as a Parlay?" |
| ✅ Tasks | "This sounds like a task — want to track it?" |
| 💛 Appreciation | "Want to save this to your appreciation journal?" |
| 🤖 Conflict | "Would it help to talk this through with the AI mediator?" |
| 💰 Budget | "A Parlay proposal can help you both agree" |

Clicking a hint navigates to the relevant creation screen.

---

## 8. AI Systems

### Edge Function

All AI calls go through a single Supabase Edge Function:
- **URL**: `[supabase-url]/functions/v1/ai-proxy`
- **Model**: Claude (Anthropic)
- **Timeout**: 45 seconds
- **Auth**: Supabase session token

### Tone Filtering Pipeline (6 Phases)

Every user-written message/note passes through this before being stored:

```
1. DECODE MESSAGE
   → Classify: Agreement / Refusal / Request / Emotional / Logistics

2. GOTTMAN FOUR HORSEMEN SCAN
   → Detect: Criticism, Contempt, Defensiveness, Stonewalling

3. NVC TRANSLATION (Nonviolent Communication)
   → Extract: Observation → Feeling → Need → Request

4. ATTACHMENT-AWARE FRAMING
   → Replace demands with invitations
   → Replace blame with vulnerability statements

5. LANGUAGE SAFETY
   → Preserve intent, prevent projection
   → No unsolicited endearments

6. CALIBRATE DELIVERY
   → Apply couple's mediator tone setting:
     - Soft 🌿 (gentle, validating)
     - Neutral ⚖️ (balanced, factual)
     - Direct 🎯 (clear, concise)
```

### ToneCoach (Real-Time Feedback)

Active on all textareas (task context, chat, counter-proposals):
- Triggers after 40+ characters typed
- 600ms debounce
- Detects Gottman Four Horsemen in real-time
- Shows inline feedback below the textarea
- Suggests rewording before sending

### AI Mediator Chat

A dedicated screen (`mediator`) where either partner can have a conversation with an AI mediator:
- System prompt includes relationship context (communication styles, love languages, active parlays, equity balance)
- Mediator tone adapts to couple's setting
- Can reference specific parlays or tasks
- Provides structured advice based on Gottman Method principles

### AI-Generated Proposals

When a parlay reaches the proposal stage:
- AI reviews both partners' stated preferences
- Considers date availability, budget, logistics
- Applies fairness principles
- Generates a structured proposal with reasoning
- Stored as JSON in the `proposal` field

### Context Building

`AIContext.buildContext()` assembles relationship data for AI calls:
- Both profiles (names, love languages)
- Communication preferences
- Active parlays and their statuses
- Recent task history
- Equity balance
- Mediator tone setting
- Family members

---

## 9. Real-Time Sync

### Supabase Realtime Channel

On bootstrap, the app subscribes to a channel named `couple-{couple_id}`:

```
Subscribed tables:
├─ negotiations (parlays)
│   └─ INSERT/UPDATE → reload parlays, show toast for new/changed events
├─ tasks
│   └─ INSERT/UPDATE → reload tasks, notify on acceptance/decline/completion
├─ chat_messages
│   └─ INSERT/UPDATE/DELETE → CoupleChat._handleRealtime()
└─ couples
    └─ UPDATE → sync settings (mediator_tone, equity)
```

### Error Handling

- `CHANNEL_ERROR` / `TIMED_OUT` → 3-second reconnect delay
- Automatic retry on reconnect
- Cleanup: `_realtime.unsubscribe()` on sign-out

### Refresh Mechanism

- Pull-to-refresh style reload on each screen
- Explicit refresh buttons on plan, tasks, and chat screens
- Full data reload calls `Store.loadParlays()`, `Store.loadTasks()`, etc.

---

## 10. Navigation & Screens

### Router Architecture

```javascript
Router.register(screenName, renderFunction, initFunction)
Router.go(screenName, params)
Router.reload()  // re-render current screen
```

- Screens are `<div class="screen" id="screen-{name}">` elements
- Only one has `.active` class at a time
- Bottom nav bar shows 5 tabs (visible when authenticated)

### Bottom Navigation Tabs

| Tab | Icon | Screen |
|-----|------|--------|
| Home | 🏠 | `home` |
| Chat | 💬 | `couple-chat` |
| Calendar | 📅 | `calendar` |
| Tasks | ✅ | `tasks` |
| Setup | ⚙️ | `setup` |

### Complete Screen Map

**Authentication:**
- `auth-entry` — Landing page (sign up / sign in / try demo)
- `auth-signup` — Email + password + name registration
- `auth-signin` — Email + password login

**Onboarding:**
- `ob-couple` — Create couple or enter invite code
- `ob-invite` — Display generated invite code with share options
- `ob-join` — Enter partner's invite code

**Main Screens:**
- `home` — Dashboard with partner bar, quick actions, urgent parlays
- `couple-chat` — Full chat interface
- `calendar` — Calendar view of confirmed events
- `tasks` — Task list with sections
- `plan` — Parlay list organized by status
- `setup` — Settings hub

**Parlay Flow:**
- `compose-pick` — Select event topic/category
- `compose` — Fill in event proposal details + dates
- `parlay-detail` — View full parlay state + actions
- `parlay-reply` — Counter-propose with new dates

**Task Flow:**
- `task-new` — Select task category
- `task-new-details` — Fill in task details + assignment
- `task-detail` — View task state + accept/decline/counter/done

**Setup & Profile:**
- `profile-edit` — Edit name, phone
- `partner-hub` — Partner connection management
- `family-setup` — Add/edit family members
- `setup-tone` — Choose mediator tone (soft/neutral/direct)
- `comm-style` — Rate 5 communication traits (1-5 scale)
- `love-languages` — View/set love languages

**Insights & Analytics:**
- `insights` — Relationship insights dashboard
- `health-dashboard` — Relationship health metrics
- `weekly-recap` — Weekly summary
- `daily-ritual` — Daily check-in
- `chat-insights` — Chat analytics
- `mediator` — AI mediator chat

---

## 11. Equity & Insights

### Equity Balance Calculation

```
myDone = tasks completed by me
theirDone = tasks completed by partner
total = myDone + theirDone

equity = {
  me: Math.round((myDone / total) * 100),
  them: Math.round((theirDone / total) * 100)
}
```

### Display Locations

- **Insights screen** — Equity bar: "Well balanced" vs. "[Name] carrying more"
- **Health dashboard** — Equity bar with context message
- **Weekly recap** — Historical equity trend
- **Coaching tips** — Rebalancing suggestions when inequality > 30%

### Achievements/Badges

Task-related achievements tracked:
- `first-task` — "Task Master" (create 1+ task)
- `five-tasks-done` — "Getting Things Done" (complete 5+ tasks)
- Additional relationship milestones tracked

---

## 12. Demo Mode

### Activation

- URL parameter: `?demo` or `?demo=B` (to play as partner B)
- "Try Demo" button on landing page → `DEMO.activate('A')`

### Behavior

- All Supabase calls intercepted — data stays in memory only
- Mock couple: "Jordan" (user A) and "Alex" (user B)
- Pre-loaded data: 3 parlays, 6 tasks, demo chat messages
- Demo bar shown at top of screen with role indicator
- Auth functions return mock data
- CRUD operations update in-memory arrays only

### Demo Data

- **Parlays**: Date night (pending), family visit (in discussion), weekend trip (resolved)
- **Tasks**: Grocery shopping, fix faucet, kids dentist, laundry, insurance, dog walking
- **Chat**: Sample conversation demonstrating tone filtering and AI hints

---

## Summary: End-to-End User Journey

```
1. DISCOVER → Landing page, "Try Demo" or sign up
2. SIGN UP → Email/password or Google OAuth
3. ONBOARD → Create couple (get invite code) OR enter partner's code
4. CONNECT → Partner signs up + enters invite code → linked
5. PLAN EVENTS → Create parlay → Partner responds → Negotiate dates → AI proposes → Confirm
6. MANAGE TASKS → Assign task → Partner accepts/counters/declines → Mark done
7. CHAT → Real-time messaging with tone filtering → AI suggests creating parlays/tasks
8. REVIEW → Equity balance, insights, weekly recap, relationship health
9. ADJUST → Change mediator tone, communication preferences, profile settings
```

Every text input (parlay notes, task context, chat messages, counter-proposals, decline reasons) passes through the AI tone-filtering pipeline before being stored and shown to the partner.
