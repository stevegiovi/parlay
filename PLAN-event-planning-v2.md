# Parlay Event Planning v2 — Complete Product Design

## A. Product Vision

Parlay's Event Planning feature lets one partner propose a shared event to the other using **structured selections that do the talking** — not paragraphs of text. The system captures who, what, when, where, and logistics through taps and toggles, optionally runs a free-text note through AI tone filtering, and gives the receiving partner clear response options. The goal: a couple can negotiate a family visit, date night, or trip in under 60 seconds, with zero friction, zero arguments, and a clean record of every decision.

**Core insight:** Most event proposals between couples are repeatable patterns (date night, family visit, kids' activity). The app should encode those patterns into structured flows so users tap instead of type.

---

## B. UX Principles

1. **Tap-first, type-optional.** Structured selections handle 80%+ of proposals. Free-text is always available but never required.
2. **Emotional safety by default.** Every free-text message passes through AI filtering before the partner sees it. Structured selections are inherently neutral.
3. **Shared visibility.** Both partners see the same event record, same status, same history. No hidden state.
4. **Mobile-native speed.** Every flow must be completable with one thumb in under 60 seconds. No scrolling through 10 fields — progressive disclosure, smart defaults.
5. **Scalable simplicity.** The list view stays clean at 5 events or 500. Status badges, filters, and sections keep things organized without overwhelming the UI.
6. **Context-aware intelligence.** The app uses Family Setup data (kids' names, family members) to populate contextual options automatically.
7. **Forgiving design.** Users can edit proposals before the partner responds, reopen declined events, and always go back.

---

## C. End-to-End User Workflow

### Partner A: Creating a Proposal

```
Plan tab → "+ New Event" button
  → Pick event type (grid of 10 types)
  → Dynamic structured fields appear based on type:
     - Who's going? (tap to select family members from Family Setup)
     - Partner involvement (going / staying home / covering kids / just needs to know)
     - When? (date picker with up to 4 options, time selectors)
     - Where? (structured options per type + optional free text)
     - Logistics notes? (optional structured: needs babysitter, transportation, budget level)
  → Optional: Add a personal note (free text, AI-filtered before partner sees it)
  → Review card showing the structured summary
  → "Send to [Partner name]" button
  → Confirmation screen with status
```

### Partner B: Receiving & Responding

```
Home tab shows inbox banner: "1 event proposal from Alex"
  → Tap → Event detail view showing:
     - Event type + icon
     - Structured summary (who, when, where, logistics) — no wall of text
     - Optional filtered note from Partner A
     - Clear action buttons:
       [✓ Accept] [📅 Suggest other times] [💬 Ask a question] [✕ Decline]

  → If Accept: Event moves to "Accepted" status. Both see confirmation.
  → If Suggest other times: Date picker appears inline. Partner A gets notified.
  → If Ask a question: Text input appears. Message is AI-filtered, sent as thread.
  → If Decline: Optional reason (AI-filtered). Event moves to "Declined" status.
```

### Message Thread (if discussion needed)

```
Event detail view shows chronological thread:
  - Original proposal (structured + optional note)
  - Partner B's response (structured action + optional note)
  - Partner A's reply
  - ... continues until resolved

Each message shows:
  - Sender avatar + name
  - Timestamp
  - Filtered text (with "Filtered by Parlay" tag)
  - Any structured updates (new dates, changed logistics)

Thread ends when event is Accepted, Declined, or Archived.
```

### Resolution

```
Accepted → Event card shows green "Confirmed" badge + confirmed date
Declined → Event card shows muted "Declined" badge
Archived → Moved to archive section, accessible but not cluttering active view
```

---

## D. UI / Screen Architecture

### D1. Plan Tab (Event List)

```
┌─────────────────────────────┐
│ ■ Plan Events          [↻]  │  ← Header (espresso bg)
│   Propose, discuss, decide  │
├─────────────────────────────┤
│ [+ New Event Proposal]      │  ← Primary CTA button
├─────────────────────────────┤
│ ⚡ NEEDS YOUR RESPONSE (2)  │  ← Orange section, only if > 0
│ ┌───────────────────────┐   │
│ │ 🌹 Date Night  · Mar 8│   │     Tap → detail
│ │ From Alex  [Reply ↩]  │   │
│ └───────────────────────┘   │
│ ┌───────────────────────┐   │
│ │ 👨‍👩‍👧 Family Visit · Mar 6│   │
│ │ From Alex  [Reply ↩]  │   │
│ └───────────────────────┘   │
├─────────────────────────────┤
│ ACTIVE (3)                  │  ← Events in discussion
│ ┌───────────────────────┐   │
│ │ ✈️ Spring Break · Mar 3│   │
│ │ In discussion 💬       │   │
│ └───────────────────────┘   │
├─────────────────────────────┤
│ UPCOMING (1)                │  ← Accepted with future dates
│ ┌───────────────────────┐   │
│ │ 🍺 Poker Night · Feb 26│   │
│ │ Confirmed ✓  Thu 7pm  │   │
│ └───────────────────────┘   │
├─────────────────────────────┤
│ [Filter ▾] [Sort ▾]        │  ← Compact filter controls
├─────────────────────────────┤
│ PAST / RESOLVED (4)         │  ← Collapsed by default
│ ┌───────────────────────┐   │
│ │ See 4 resolved events ▾│   │
│ └───────────────────────┘   │
└─────────────────────────────┘
```

**Empty state (no events):**
```
┌─────────────────────────────┐
│         📅                   │
│   No events yet              │
│   Propose a date night,      │
│   family visit, or trip.     │
│                              │
│   [+ New Event Proposal]     │
└─────────────────────────────┘
```

### D2. Create Proposal Flow (Multi-step, but fast)

**Step 1: Pick event type** (grid, same as current topic-grid)

**Step 2: Structured details** (adapts per type — see Section E)
```
┌─────────────────────────────┐
│ ‹ Back    🌹 Date Night     │
├─────────────────────────────┤
│ QUICK SETUP                  │
│                              │
│ Who's going?                 │
│ [Just us ✓] [With kids]     │
│ [Double date] [Family]      │
│                              │
│ When?                        │
│ ┌─ Option 1 ─────────────┐ │
│ │ [Mar 14 ▾] [7:00 PM ▾] │ │
│ └─────────────────────────┘ │
│ [+ Add another option]      │
│                              │
│ Where?                       │
│ [Restaurant] [Home ✓]       │
│ [Activity] [Surprise]       │
│                              │
│ ─── Optional note ───        │
│ ┌─────────────────────────┐ │
│ │ Add a personal note...  │ │
│ │ (Parlay filters before  │ │
│ │  sending)               │ │
│ └─────────────────────────┘ │
│                              │
│ [Send to Jordan →]          │
└─────────────────────────────┘
```

**Key UX details:**
- Structured fields use pill-button selectors (not dropdowns) for speed
- Date/time uses native pickers (one tap to open)
- "Add another option" lets user propose up to 4 date/time slots
- Family member selection pulls from Family Setup (kids by name)
- Note field is collapsed by default, expandable with "Add a note" link
- Title is auto-generated from structured data: "Date Night — Fri Mar 14, 7pm"

### D3. Event Detail View

```
┌─────────────────────────────┐
│ ‹ Plans              [⋯]   │
├─────────────────────────────┤
│ 🌹 Date Night               │
│ Mar 8 · Alex proposed       │
│ ┌─ Proposed ──────── ○ ─┐  │  ← Status pill
│ └───────────────────────┘   │
├─────────────────────────────┤
│ ┌─ DETAILS ──────────────┐  │
│ │ Who:    Just us two     │  │
│ │ Where:  Restaurant      │  │
│ │ When:                   │  │
│ │  ● Fri Mar 14, 7pm     │  │
│ │  ○ Sat Mar 15, 6:30pm  │  │
│ └─────────────────────────┘  │
│                              │
│ ┌─ NOTE FROM ALEX ────────┐ │
│ │ "I'd love to try that   │ │
│ │  new Italian place."    │ │
│ │  ✓ Filtered by Parlay   │ │
│ └─────────────────────────┘ │
├─────────────────────────────┤
│ ┌─ THREAD ────────────────┐ │  ← Only if messages exist
│ │ Jordan · Mar 8, 6:12pm  │ │
│ │ "Sounds great! Can we   │ │
│ │  do 7:30 instead?"      │ │
│ │                          │ │
│ │ Alex · Mar 8, 6:45pm    │ │
│ │ "7:30 works perfectly."  │ │
│ └──────────────────────────┘ │
├─────────────────────────────┤
│ ── ACTIONS ──                │
│ [✓ Accept] [💬 Reply]       │
│ [📅 Suggest times] [✕ Decline]
└─────────────────────────────┘
```

### D4. Filter/Sort Controls (bottom sheet)

```
┌─────────────────────────────┐
│ Filter & Sort                │
├─────────────────────────────┤
│ STATUS                       │
│ [All] [Needs me ✓] [Active] │
│ [Accepted] [Declined]        │
│                              │
│ TYPE                         │
│ [All ✓] [🌹] [👨‍👩‍👧] [✈️]      │
│ [🍺] [🍷] [🏡] [👧] [💬]    │
│                              │
│ INVOLVES                     │
│ [Anyone ✓] [Kids] [Emma]    │
│ [Liam] [In-laws]            │
│                              │
│ SORT                         │
│ ○ Newest  ● Upcoming        │
│ ○ Oldest  ○ Unresolved first │
│                              │
│ [Apply]                      │
└─────────────────────────────┘
```

### D5. Family Setup Screen (new, under Setup tab)

```
┌─────────────────────────────┐
│ ‹ Setup                     │
│ Family Setup                 │
├─────────────────────────────┤
│ HOUSEHOLD                    │
│ ┌───────────────────────┐   │
│ │ 👶 Emma · age 4        │ [✎]
│ │ 👦 Liam · age 7        │ [✎]
│ └───────────────────────┘   │
│ [+ Add child]               │
├─────────────────────────────┤
│ EXTENDED FAMILY              │
│ ┌───────────────────────┐   │
│ │ Grandma Sue (Alex's)   │ [✎]
│ │ Grandpa Joe (Alex's)   │ [✎]
│ │ Aunt Maria (Jordan's)  │ [✎]
│ └───────────────────────┘   │
│ [+ Add family member]       │
├─────────────────────────────┤
│ REGULAR CONTACTS             │
│ ┌───────────────────────┐   │
│ │ Babysitter: Sarah      │ [✎]
│ └───────────────────────┘   │
│ [+ Add contact]             │
└─────────────────────────────┘
```

### D6. Personal Profile Screen (new, under Setup tab)

```
┌─────────────────────────────┐
│ ‹ Setup                     │
│ My Profile                   │
├─────────────────────────────┤
│ ┌───────────────────────┐   │
│ │ Display name            │  │
│ │ [Alex                 ] │  │
│ │                         │  │
│ │ Email                   │  │
│ │ alex@email.com (locked) │  │
│ │                         │  │
│ │ Phone (optional)        │  │
│ │ [(555) 123-4567       ] │  │
│ └─────────────────────────┘  │
│                              │
│ [Save Changes]               │
└─────────────────────────────┘
```

---

## E. Event Types and Dynamic Field Logic

### Event Type Catalog

| Type | Icon | Structured Fields |
|------|------|-------------------|
| Date Night | 🌹 | who (just-us / double / family), where (restaurant / home / activity / surprise), when, dress code (casual/nice), needs babysitter? |
| Boys' Night | 🍺 | who's going (just me), partner role (home / covering kids / also out), when, where (general), needs babysitter? |
| Girls' Night | 🍷 | who's going (just me), partner role (home / covering kids / also out), when, where (general), needs babysitter? |
| Family Visit | 👨‍👩‍👧 | whose family (mine / theirs / both), direction (they come / we go / meet), who from household goes (select family members), when, duration (day / overnight / multi-day), needs from partner |
| Trip / Vacation | ✈️ | who's going (select family members), type (weekend / week / day trip), when (date range), destination type (beach / city / nature / visiting someone), budget comfort (budget / moderate / splurge), needs from partner |
| Hosting | 🏡 | format (dinner / drinks / BBQ / game night / party), guest count (small / medium / large), who's invited (friends / family / mixed), prep split (share / I handle / plan together), when, kids included? |
| Playdate | 👧 | which kids (select from Family Setup), who's supervising (me / partner / both / other parent), when, where (our place / their place / park / activity), partner needs to know? |
| Kids' Activity | 🎒 | which kids, activity type (sports / lessons / school event / birthday party), who takes them, recurring?, when, transportation needs |
| Household Event | 🔧 | type (repair / service visit / delivery / project), when, who needs to be home, estimated time, any prep needed |
| Something Else | 💬 | who's involved (select family members), when, free-text description (required for this type) |

### Dynamic Field Rendering Rules

Each field type renders as a specific UI component:

- **Select-one:** Pill buttons in a horizontal wrap (e.g., "who's going")
- **Select-many:** Pill buttons with multi-select (e.g., "which kids")
- **Date/time:** Native date input + time dropdown, grouped as "Option 1, 2, 3, 4"
- **Date range:** Start date + end date (for trips)
- **Yes/No:** Two pill buttons
- **Free text:** Expandable textarea with "Add a note" trigger

### Family-Aware Fields

When Family Setup has kids defined:
- "Which kids?" shows each child by name as a tappable pill
- "Needs babysitter?" only appears if kids exist and neither partner is going

When Family Setup has extended family:
- Family Visit "who's coming" can show saved family members

---

## F. Data Model / Schema Recommendations

### F1. New Tables

#### `family_members` — Household context
```sql
CREATE TABLE family_members (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  couple_id UUID NOT NULL REFERENCES couples(id),
  name TEXT NOT NULL,
  role TEXT NOT NULL,        -- 'child', 'extended', 'contact'
  relation TEXT,             -- 'son', 'daughter', 'grandparent', 'babysitter', etc.
  belongs_to TEXT,           -- 'both', spouse_1_id, or spouse_2_id (whose side)
  birth_date DATE,           -- for kids (age calculation)
  notes TEXT,                -- optional
  sort_order INT DEFAULT 0,
  created_at TIMESTAMPTZ DEFAULT now(),
  deleted_at TIMESTAMPTZ     -- soft delete
);
-- RLS: couple members only
```

#### `event_messages` — Thread messages for events
```sql
CREATE TABLE event_messages (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  event_id UUID NOT NULL REFERENCES negotiations(id),
  couple_id UUID NOT NULL REFERENCES couples(id),
  sender_uid UUID NOT NULL,
  message_type TEXT NOT NULL DEFAULT 'text',
    -- 'text', 'date_proposal', 'date_accept', 'status_change', 'system'
  raw_content TEXT,          -- what user typed (private)
  filtered_content TEXT,     -- AI-filtered version (shown to partner)
  structured_data JSONB,     -- for date proposals, selections, etc.
  created_at TIMESTAMPTZ DEFAULT now(),
  deleted_at TIMESTAMPTZ
);
-- RLS: couple members only
-- Index: (event_id, created_at)
```

### F2. Modified Table: `negotiations` → rename columns conceptually to "events"

Keep the existing `negotiations` table but add/modify columns:

```sql
ALTER TABLE negotiations ADD COLUMN IF NOT EXISTS
  structured_selections JSONB,     -- the structured field answers
  auto_title TEXT,                  -- system-generated title from selections
  event_date_start DATE,           -- confirmed/proposed primary date (for sorting)
  event_date_end DATE,             -- for multi-day events
  involves JSONB,                  -- array of family_member IDs involved
  message_count INT DEFAULT 0,     -- denormalized for list badge
  last_activity_at TIMESTAMPTZ DEFAULT now();  -- for sort-by-recent
```

**Revised status enum for `negotiations.status`:**
```
'proposed'          -- just sent, awaiting partner
'awaiting_response' -- partner has seen it (optional, if read-receipts added later)
'in_discussion'     -- at least one message exchanged
'accepted'          -- partner accepted
'declined'          -- partner declined
'archived'          -- manually archived by either partner
```

**Keep existing columns that still work:**
- `id`, `couple_id`, `topic_id`, `title`, `raw_input`, `filtered_input`
- `intent_context`, `date_options`, `status`, `created_by`
- `partner_raw_reply`, `partner_filtered_reply` — still used for the first reply
- `counter_dates`, `confirmed_date`, `proposal`, `resolution`
- `archived`, `deleted_at`, `created_at`

### F3. Modified Table: `profiles` — Add personal fields

```sql
ALTER TABLE profiles ADD COLUMN IF NOT EXISTS
  phone TEXT,
  avatar_color TEXT DEFAULT '#C4714A';
```

### F4. Indexes for Scale

```sql
CREATE INDEX idx_negotiations_couple_status
  ON negotiations(couple_id, status) WHERE deleted_at IS NULL;
CREATE INDEX idx_negotiations_couple_activity
  ON negotiations(couple_id, last_activity_at DESC) WHERE deleted_at IS NULL;
CREATE INDEX idx_event_messages_event
  ON event_messages(event_id, created_at);
CREATE INDEX idx_family_members_couple
  ON family_members(couple_id) WHERE deleted_at IS NULL;
```

---

## G. State Logic

### State Machine

```
                    ┌─────────────────────────────────┐
                    │                                 │
  [Create] ──→ PROPOSED ──→ ACCEPTED ──→ ARCHIVED    │
                 │   │         ↑                      │
                 │   │         │                      │
                 │   ├──→ IN_DISCUSSION ──┤           │
                 │   │         ↑    │     │           │
                 │   │         │    │     ↓           │
                 │   │         │    ├──→ ACCEPTED ──→ ARCHIVED
                 │   │         │    │
                 │   │         │    ├──→ DECLINED ──→ ARCHIVED
                 │   │         │    │
                 │   │         │    └──→ (loop back to IN_DISCUSSION)
                 │   │         │
                 │   ├──→ DECLINED ──→ ARCHIVED
                 │   │
                 │   └──→ ARCHIVED (by creator before response)
                 │
                 └──→ [Edit] (only before partner responds)
```

### Transition Rules

| From | Action | To | Who can do it |
|------|--------|----|---------------|
| proposed | Partner accepts | accepted | Recipient only |
| proposed | Partner declines | declined | Recipient only |
| proposed | Partner sends message | in_discussion | Recipient only |
| proposed | Partner suggests times | in_discussion | Recipient only |
| proposed | Creator edits | proposed (updated) | Creator only |
| proposed | Creator archives | archived | Creator only |
| in_discussion | Either accepts | accepted | Either partner |
| in_discussion | Either declines | declined | Either partner |
| in_discussion | Either sends message | in_discussion | Either partner |
| in_discussion | Either suggests times | in_discussion | Either partner |
| accepted | Either archives | archived | Either partner |
| declined | Either archives | archived | Either partner |
| declined | Creator reopens | proposed | Creator only |
| archived | Either reopens | last active status | Either partner |

### "Needs my response" Logic

An event needs Partner X's response when:
1. Status is `proposed` AND Partner X is the recipient (not the creator)
2. Status is `in_discussion` AND the most recent message/action was from the other partner
3. Status is `in_discussion` AND a new date proposal was sent by the other partner

This is computed client-side from `last_activity_by` field (stored on negotiations) or from the latest `event_messages` row.

---

## H. AI Filtering Integration

### Where AI Runs

| Trigger | AI Action | Blocking? |
|---------|-----------|-----------|
| Creator adds a personal note when proposing | Tone filter on note text | Yes — show spinner, send filtered version |
| Recipient sends a reply message | Tone filter on reply text | Yes — show spinner |
| Recipient sends a decline reason | Tone filter on reason | Yes — show spinner |
| Any threaded message | Tone filter | Yes — show spinner |
| Creator submits a very long structured event | (optional) AI summary of structured data | No — best effort |

### Where AI Does NOT Run

- Structured selections (they're inherently neutral — "Who's going: Just us" needs no filtering)
- Status changes (accept, decline, archive — these are actions, not messages)
- Date/time proposals (structured data, no text)
- System-generated messages ("Alex accepted the event")

### AI Flow for Free-Text

```
User types raw message
  → User taps "Send"
  → Button shows spinner: "Filtering..."
  → AI.call() with tone filter prompt + message
  → On success: Save raw (private) + filtered (visible) to event_messages
  → On failure: Save raw as-is with "⚠️ Filter unavailable" tag
  → Recipient sees filtered version only
```

### Demo Mode AI

The existing `DemoRewriter` handles all of this locally — no changes needed. It already intercepts `AI.call()` and returns local rewrites for tone filtering.

---

## I. Scalability / Filtering Design

### List Organization Strategy

**Smart sections** (auto-organized, no user effort):

1. **Needs your response** — events waiting for this user's action. Always on top if > 0.
2. **Active** — in_discussion or proposed-by-me events. Sorted by last activity.
3. **Upcoming** — accepted events with future dates. Sorted by event date ascending.
4. **Past** — accepted events with past dates. Collapsed by default.
5. **Resolved** — declined events. Collapsed by default.

**User-activated filters** (bottom sheet, persistent until cleared):

- Status: Needs me / Active / Accepted / Declined / All
- Type: Any / specific event type icons
- Involves: Any / specific family member
- Date range: This week / This month / Custom
- Sort: Recent activity / Upcoming date / Oldest first

### Performance at Scale

- **Pagination:** Load 25 events initially, "Load more" button at bottom
- **Denormalized counts:** `message_count` and `last_activity_at` on negotiations table so list view doesn't need to join messages
- **Client-side filtering:** For the first 100 events, filter/sort entirely in-memory (Store already holds all data). Beyond that, push filters to Supabase query.
- **Archived separation:** Archived events are NOT loaded by default. A separate "View archive" link loads them on demand.
- **Index strategy:** Composite index on `(couple_id, status, last_activity_at)` covers the main list query efficiently.

### Filter UI at Scale

At < 10 events: No filter controls shown (unnecessary clutter)
At 10-30 events: Show a single "Filter" button that opens the sheet
At 30+ events: Show inline status tabs (Needs me | Active | All) + filter button for advanced

---

## J. Build Recommendation

### Architecture Approach

Keep the single-file SPA architecture. The changes are additive:

1. Add new CSS for pill-button selectors, thread messages, and family setup
2. Add new JS modules: `FamilyStore` (manages family_members), `EventMessages` (manages thread)
3. Modify existing modules: `Store` (add family/messages loading), `DEMO` (add mock family data)
4. Add new screens: `family-setup`, `profile-edit`, enhanced `compose` flow, enhanced `parlay-detail`
5. Modify existing screens: `plan` (new list organization), `setup` (add family/profile links)
6. Add Supabase migrations for new tables and columns

### Implementation Priority Order

#### Phase 1: Foundation (do first)
1. **Family Setup table + screen** — Create `family_members` table, build the Setup screen, wire CRUD.
2. **Profile enhancements** — Add phone field to profiles, build profile edit screen.
3. **Schema migration** — Add `structured_selections`, `auto_title`, `event_date_start`, `last_activity_at`, `involves` columns to negotiations.
4. **Demo mode updates** — Add mock family members to DEMO data.

#### Phase 2: Structured Compose (core feature)
5. **Pill-button UI component** — Build reusable pill-selector (single + multi-select variants).
6. **Dynamic compose flow** — Replace current free-text-first compose with structured-selections-first flow. Keep free text as optional note.
7. **Event type field configs** — Define the structured fields per event type (the logic from Section E).
8. **Auto-title generation** — Generate titles like "Date Night — Fri Mar 14" from structured data.
9. **Family-aware fields** — Wire "which kids" and "family members" fields to pull from FamilyStore.

#### Phase 3: Enhanced Response Flow
10. **Response actions** — Replace current reply screen with clear action buttons: Accept, Decline, Suggest times, Send message.
11. **Event messages table** — Create `event_messages` table, build thread UI in detail view.
12. **Message threading** — Show chronological thread of all exchanges within an event.
13. **Status transitions** — Implement the full state machine from Section G.

#### Phase 4: List & Scale
14. **Smart list sections** — Reorganize Plan tab into Needs me / Active / Upcoming / Past sections.
15. **Filter/sort UI** — Build the filter bottom sheet with status, type, and family filters.
16. **Badge updates** — Update Badges to reflect new status categories.
17. **Realtime for messages** — Subscribe to `event_messages` changes for live thread updates.

#### Phase 5: Polish
18. **Edit-before-response** — Let creator edit a proposal before partner responds.
19. **Reopen flow** — Let creator reopen declined events.
20. **Archive management** — Separate archive loading, archive/unarchive UI.
21. **Pagination** — Add "Load more" for couples with 50+ events.

### Migration Safety

All schema changes should be **additive** (new columns, new tables). No existing columns are dropped or renamed. The current `negotiations` table continues to work — new fields default to null. The app detects the presence of `structured_selections` to determine whether to show the new or legacy compose flow, providing backwards compatibility during the transition.

### Estimated Scope

- Phase 1: ~300 lines new code
- Phase 2: ~600 lines new code, ~200 lines modified
- Phase 3: ~500 lines new code, ~300 lines modified
- Phase 4: ~400 lines new code, ~200 lines modified
- Phase 5: ~300 lines new code

Total: ~2,300 new lines + ~700 modified lines. The file grows from ~3,950 to ~6,250 lines — still manageable as a single-file SPA.

---

## Summary: Recommended Next Steps

1. **Start with Family Setup** — it's the data foundation everything else builds on
2. **Build the structured compose flow** — this is the core product differentiator
3. **Layer in the response actions and threading** — turns one-way proposals into conversations
4. **Add list organization and filters** — scales the experience as usage grows
5. **Polish edge cases** — edit, reopen, archive, pagination

Each phase ships a working improvement. No phase depends on a later phase. The couple can use each increment immediately.
