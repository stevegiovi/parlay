# Parlay Tasks v2 — Comprehensive Design Plan

## A. Vision & Philosophy

Tasks in Parlay are **quick asks between partners** — not project management. The vibe is a shared household clipboard, not Jira. Tasks should be:

- **Fast to create** — 2 taps to assign a common task
- **Low friction to respond** — Accept with one tap, done with one tap
- **Transparent** — both partners see everything, no hidden lists
- **Respectful** — AI filters any tension from context/decline reasons
- **Practical** — built around what real couples actually ask each other

---

## B. The 30 Most Common Couple Tasks

Organized into categories, these are the quick-tap presets:

### 🛒 Errands & Shopping
1. Grab groceries
2. Pick up prescriptions
3. Return an item / make a return
4. Drop off dry cleaning
5. Get gas / charge the car

### 🏠 Household
6. Do the dishes
7. Vacuum / mop the floors
8. Do laundry (wash/fold/put away)
9. Take out the trash & recycling
10. Clean the bathroom(s)
11. Tidy the kitchen
12. Organize the garage / closet

### 👧 Kids
13. Pack lunches
14. Do school pickup / dropoff
15. Schedule a doctor / dentist appointment
16. Help with homework
17. Give the kids a bath
18. Sign permission slip / school form
19. Arrange a playdate

### 🔧 Fix & Repair
20. Fix the leaky faucet
21. Change a lightbulb
22. Mow the lawn / yard work
23. Take car in for service
24. Hang shelves / pictures

### 📋 Admin & Finance
25. Pay a bill
26. Call insurance / provider
27. Schedule an appointment
28. File paperwork / taxes
29. Update a subscription

### 🐾 Pets
30. Walk the dog / feed the pets

---

## C. Category System (Expanded)

```
TASK_CATEGORIES_V2 = [
  { id: 'errands',     icon: '🛒', label: 'Errands',      color: '#E8B88A' },
  { id: 'household',   icon: '🏠', label: 'Household',    color: '#C4714A' },
  { id: 'kids',        icon: '👧', label: 'Kids',          color: '#7A6A9B' },
  { id: 'fix',         icon: '🔧', label: 'Fix & Repair',  color: '#6A8CAA' },
  { id: 'admin',       icon: '📋', label: 'Admin',         color: '#9B8880' },
  { id: 'pets',        icon: '🐾', label: 'Pets',          color: '#7A9E8A' },
  { id: 'selfcare',    icon: '💆', label: 'Self-care',     color: '#D4A5A5' },
  { id: 'other',       icon: '📌', label: 'Other',         color: '#9B8880' },
]
```

Added **Self-care** (schedule haircut, gym, etc.) and **color** for visual badges.

---

## D. Quick-Tap Presets (TASK_PRESETS)

Each category has 4-6 presets — tapping one pre-fills the title and category:

```
TASK_PRESETS = {
  errands: [
    { title: 'Grab groceries',          icon: '🛒' },
    { title: 'Pick up prescriptions',   icon: '💊' },
    { title: 'Return an item',          icon: '📦' },
    { title: 'Drop off dry cleaning',   icon: '👔' },
    { title: 'Get gas',                 icon: '⛽' },
  ],
  household: [
    { title: 'Do the dishes',           icon: '🍽️' },
    { title: 'Do laundry',              icon: '🧺' },
    { title: 'Vacuum / mop',            icon: '🧹' },
    { title: 'Take out trash',          icon: '🗑️' },
    { title: 'Clean the bathroom',      icon: '🚿' },
    { title: 'Tidy the kitchen',        icon: '🧽' },
  ],
  kids: [
    { title: 'Pack lunches',            icon: '🥪' },
    { title: 'School pickup',           icon: '🚗' },
    { title: 'Schedule doctor appt',    icon: '🩺' },
    { title: 'Help with homework',      icon: '📚' },
    { title: 'Give kids a bath',        icon: '🛁' },
    { title: 'Sign school form',        icon: '✍️' },
  ],
  fix: [
    { title: 'Fix the leaky faucet',    icon: '🚰' },
    { title: 'Change a lightbulb',      icon: '💡' },
    { title: 'Mow the lawn',            icon: '🌿' },
    { title: 'Take car in for service', icon: '🚙' },
    { title: 'Hang shelves / pictures', icon: '🖼️' },
    { title: 'Yard work',               icon: '🌳' },
  ],
  admin: [
    { title: 'Pay a bill',              icon: '💳' },
    { title: 'Call insurance',          icon: '📞' },
    { title: 'Schedule appointment',    icon: '📅' },
    { title: 'File paperwork',          icon: '📄' },
    { title: 'Update subscription',     icon: '🔄' },
  ],
  pets: [
    { title: 'Walk the dog',            icon: '🐕' },
    { title: 'Feed the pets',           icon: '🥣' },
    { title: 'Vet appointment',         icon: '🏥' },
    { title: 'Groom / bathe pet',       icon: '🐩' },
  ],
  selfcare: [
    { title: 'Schedule haircut',        icon: '💇' },
    { title: 'Gym / workout',           icon: '🏋️' },
    { title: 'Book massage / spa',      icon: '💆' },
  ],
}
```

---

## E. Screen Architecture

### E1. Tasks List (main tab)

```
┌──────────────────────────────────┐
│ ⭐ Tasks & To-Dos        ↻      │  ← espresso header
│                                  │
├──────────────────────────────────┤
│  [+ Assign a Task]              │
│                                  │
│  ⚡ NEEDS YOUR ACTION (2)        │  ← orange highlight section
│  ┌────────────────────────────┐ │
│  │ 🛒 Grab groceries          │ │  ← task card with category icon
│  │    From Jordan · Due today │ │
│  │    ⚡ Accept or respond     │ │
│  └────────────────────────────┘ │
│  ┌────────────────────────────┐ │
│  │ 🔧 Fix leaky faucet        │ │
│  │    From Jordan · No due    │ │
│  │    ⚡ Accept or respond     │ │
│  └────────────────────────────┘ │
│                                  │
│  ▾ My Tasks (3)                 │  ← collapsible section
│  ┌────────────────────────────┐ │
│  │ 🧺 Do laundry     [✓ Done] │ │  ← inline "Done" quick button
│  │    Due Fri · Accepted       │ │
│  └────────────────────────────┘ │
│  ┌────────────────────────────┐ │
│  │ 📚 Help with homework      │ │
│  │    Due today · Accepted     │ │
│  └────────────────────────────┘ │
│                                  │
│  ▾ I Assigned (2)               │
│  ┌────────────────────────────┐ │
│  │ 🩺 Schedule doctor appt    │ │
│  │    To Jordan · Pending      │ │
│  └────────────────────────────┘ │
│                                  │
│  ▸ Completed (4)                │  ← collapsed by default
└──────────────────────────────────┘
```

**Key changes from v1:**
- "Needs Your Action" section at top (like Plan's "Needs Me")
- Inline **[✓ Done]** button on accepted tasks assigned to me — one tap completion
- Due date displayed with urgency colors (overdue = red, today = orange, this week = normal)
- Collapsible sections with counts
- Category icon + color on each card

### E2. Task New — Two-step flow

**Step 1: Pick or type**

```
┌──────────────────────────────────┐
│ ‹ Back    New Task               │
│                                  │
│  What needs doing?               │
│  Tap a common task or type your  │
│  own below.                      │
│                                  │
│  🛒 Errands                      │  ← category tabs (horizontal scroll)
│  ──────────────────────────────  │
│  [🛒 Grab groceries ]           │  ← preset pill buttons
│  [💊 Pick up prescriptions]     │
│  [📦 Return an item    ]        │
│  [👔 Drop off dry cleaning]     │
│  [⛽ Get gas            ]        │
│                                  │
│  ── or ──                        │
│                                  │
│  [ Type your own task...      ]  │  ← text input fallback
│                                  │
│  Next →                          │
└──────────────────────────────────┘
```

**Step 2: Details**

```
┌──────────────────────────────────┐
│ ‹ Back    🛒 Grab groceries      │
│                                  │
│  Assign to                       │
│  [  Jordan  ] [  Myself  ]       │  ← pill buttons
│                                  │
│  📅 Due by (optional)            │
│  [ date picker             ]     │
│                                  │
│  ⚡ Priority                     │
│  [ Normal ] [ High ⚡]           │
│                                  │
│  💬 Add details (optional)       │  ← collapsible
│  ┌───────────────────────────┐  │
│  │ We need milk, eggs, and   │  │
│  │ fruit for the week...     │  │
│  └───────────────────────────┘  │
│  🔒 Filtered before sending      │
│                                  │
│  [    Assign Task →     ]        │
└──────────────────────────────────┘
```

### E3. Task Detail (enhanced)

```
┌──────────────────────────────────┐
│ ‹ Tasks                    ↻  ⋯ │
│                                  │
│  🛒                              │
│  Grab groceries                  │
│  From Alex → Jordan              │
│                                  │
│  [  ⚡ Your turn  ]              │  ← status chip
│                                  │
│  📅 Due: Friday, Mar 14          │
│  ⚡ Priority: Normal             │
│                                  │
│  Context                         │
│  ┌───────────────────────────┐  │
│  │ We need milk, eggs, bread │  │
│  │ and fruit for the week.   │  │
│  │ ✓ Filtered by Parlay      │  │
│  └───────────────────────────┘  │
│                                  │
│  ┌───────────────────────────┐  │  ← action buttons
│  │ [  ✓ Accept             ] │  │
│  │ [  ↩ Suggest Changes    ] │  │
│  │ [  ✕ Decline            ] │  │
│  └───────────────────────────┘  │
│                                  │
│  OR (if accepted, I'm assignee): │
│  ┌───────────────────────────┐  │
│  │ [  ✓ Mark Done          ] │  │
│  └───────────────────────────┘  │
│                                  │
│  🗑 Delete task                   │
└──────────────────────────────────┘
```

**Key changes from v1:**
- Due date displayed prominently with urgency indicator
- Priority badge
- Manage menu (⋯) for edit/delete
- Same action flow as v1 (works well) but with improved layout

### E4. Task Sent (confirmation)

```
┌──────────────────────────────────┐
│                                  │
│            📋                    │
│     Task assigned                │
│                                  │
│  Jordan will see your task with  │
│  any notes filtered for tone.    │
│                                  │
│  🛒 Grab groceries               │
│                                  │
│  [  View All Tasks →  ]         │
│  [  Back to Home      ]         │
└──────────────────────────────────┘
```

---

## F. Enhanced Task Data Model

```javascript
{
  id: string,
  title: string,
  categoryId: string,           // from TASK_CATEGORIES_V2
  presetIcon: string|null,      // icon from preset (if selected)
  assignedTo: 'me'|'them',
  createdBy: 'me'|'them',
  status: 'pending_response'|'accepted'|'declined'|'countered'|'done',
  pendingResponseFrom: 'me'|'them'|null,
  priority: 'normal'|'high',
  dueDate: string|null,         // ISO date

  // Context (optional free text, AI-filtered)
  context: string,
  rawContext: string,
  filteredContext: string,

  // Counter-proposal
  counterCount: number,
  counterMessage: string|null,
  filteredCounter: string|null,

  // Decline
  declineReason: string|null,
  filteredDecline: string|null,

  // Completion tracking
  completedAt: string|null,     // NEW: ISO timestamp when marked done
  completedBy: string|null,     // NEW: UID of who marked it done
}
```

**Changes from v1:**
- Added `presetIcon` — stores the unique icon if a preset was used (distinct from category icon)
- Added `completedAt` and `completedBy` for completion tracking
- No structural changes to the DB schema needed (we store these in existing columns or as JSON)

---

## G. Task Card Design

Each task card shows:

```
┌─────────────────────────────────────────┐
│  🛒  Grab groceries            [✓ Done] │  ← icon · title · quick action
│      Due Fri · Accepted · Alex          │  ← metadata line
└─────────────────────────────────────────┘
```

- **Left icon**: Category icon (or preset icon if available)
- **Title**: Bold, truncated if too long
- **Due date**: Colored by urgency
  - Overdue: red text `Overdue!`
  - Due today: orange text `Due today`
  - This week: normal text `Due Fri`
  - No due date: gray text `No due date`
- **Status chip**: Compact inline status
- **Quick action button**: Only on accepted tasks assigned to me
  - `[✓ Done]` button — completes in one tap without opening detail

---

## H. Interaction Flows

### H1. Create Task (happy path)
1. Tap "+ Assign a Task" → task-new screen (step 1)
2. See category tabs → tap a category → see presets
3. Tap a preset (e.g., "Grab groceries") → auto-fills title + category
4. OR type custom title in text input → "Next"
5. Step 2: assign to partner or self, set optional due date, priority, notes
6. Tap "Assign Task →" → AI filters notes → saves → task-sent confirmation

### H2. Respond to Task
1. See "Needs Your Action" badge on Tasks tab
2. Tap the task card → task-detail
3. Options:
   - **Accept**: One tap → status = accepted, assignee can now complete
   - **Suggest Changes**: Shows inline form → AI filters → status = countered
   - **Decline**: Shows inline form → AI filters → status = declined

### H3. Complete Task
- **From list**: Tap inline `[✓ Done]` button → immediate completion
- **From detail**: Tap "Mark Done" button → status = done

### H4. Edit Task (NEW)
- Tap ⋯ menu on task-detail → "Edit" → task-edit screen
- Can change: title, category, due date, priority, context
- Cannot change: assignment (must delete and recreate)

---

## I. Smart Features

### I1. Due Date Intelligence
- Tasks overdue get a red `⚠️ Overdue` badge
- Tasks due today get an orange `📅 Today` indicator
- Sorting: overdue first, then due today, then by due date ascending, then no-due-date last

### I2. Completion Celebrations
- When marking done: brief toast "Task done ✓"
- Completed section shows when task was done

### I3. Auto-categorization
- When user types a custom task, auto-detect category from keywords:
  - "groceries", "pick up", "buy" → errands
  - "laundry", "dishes", "clean", "vacuum" → household
  - "kids", "school", "homework", "lunch" → kids
  - "fix", "repair", "mow", "install" → fix
  - "call", "pay", "bill", "insurance" → admin
  - "dog", "cat", "pet", "vet" → pets
  - Falls through to "other"

---

## J. Demo Data (Updated)

6 mock tasks showcasing various states:

1. **"Grab groceries"** — 🛒 errands, assigned to me, pending_response (needs my action), due today
2. **"Fix leaky kitchen faucet"** — 🔧 fix, assigned to me by them, accepted, no due
3. **"Schedule dentist for kids"** — 🩺 kids, assigned to them by me, pending_response (waiting), due Mar 15
4. **"Do laundry"** — 🧺 household, assigned to me by me, accepted, due Friday
5. **"Call insurance about claim"** — 📋 admin, assigned to them, accepted, priority high, due tomorrow
6. **"Walk the dog"** — 🐕 pets, assigned to them by me, done

---

## K. Build Phases

### Phase 1: Data & Constants
- Add `TASK_CATEGORIES_V2` with colors
- Add `TASK_PRESETS` with all 30+ presets
- Add `autoCategorize(title)` helper
- Add `dueLabel(dueDate)` helper for urgency display
- Update mock data to 6 tasks with new fields

### Phase 2: Task List Screen (replace)
- Smart sections: Needs Action, My Tasks, I Assigned, Completed
- Collapsible sections with counts
- Inline `[✓ Done]` quick button on accepted tasks
- Due date urgency colors
- Category icon + color on cards
- Sorting: overdue → today → upcoming → no-date

### Phase 3: Task New Screen (replace)
- Step 1: Category tab bar + preset pill grid + custom input
- Step 2: Assign to, due date, priority, optional filtered context
- Task-sent confirmation screen

### Phase 4: Task Detail Screen (enhance)
- Better layout with due date and priority display
- Manage menu (⋯) for delete
- Same action flows (accept/counter/decline/done) — already solid
- Show completion timestamp for done tasks

### Phase 5: Polish
- Auto-categorize custom titles
- Demo data update
- Badge count verification
- Syntax validation, commit, push
