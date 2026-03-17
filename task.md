# UI Overhaul and Savings Goal Progress

## 1. Scope and Constraints

Implement a persistent Savings Goal feature and a full dashboard visual overhaul.

1. Keep existing Flask behavior working.
2. Keep existing routes and response shapes unchanged unless adding new goal-related behavior.
3. Users can have only one active goal at a time.
4. Existing money formatting logic must be reused for all displayed currency values.
5. Jinja server-rendered flow is preferred over building a separate frontend API layer.

## 2. Savings Goal Feature Requirements

### 2.1 Goal States and Lifecycle

Each goal has one status: `active`, `completed`, or `cancelled`.

1. New goal starts as `active`.
2. A goal switches to `completed` only when the user explicitly clicks a Complete Goal button.
3. A goal switches to `cancelled` only when the user explicitly clicks a Cancel Goal button.
4. Reaching or exceeding target does not auto-complete the goal.
5. Updating an active goal overwrites that same active row in place (do not create a second active goal row).
6. When a goal becomes `completed` or `cancelled`, save final net savings and end timestamp.

### 2.2 Database Schema

Create a `financial_goal` table (or SQLAlchemy model `FinancialGoal`) with:

1. `id`: integer primary key.
2. `goal_name`: string, required.
3. `target_amount`: decimal, required, positive.
4. `status`: string enum-like value in `active`, `completed`, `cancelled`.
5. `final_net_savings`: decimal, nullable; required at completion/cancellation time.
6. `created_at`: datetime, required.
7. `ended_at`: datetime, nullable; set when status becomes completed/cancelled.

Notes:

1. Use decimal-safe types for monetary data.
2. Enforce one active goal at a time in app logic.

Example object:

`FinancialGoal(id=1, goal_name="New Laptop", target_amount=10000.00, status="active", final_net_savings=None, created_at=datetime.utcnow(), ended_at=None)`

### 2.3 Validation Rules

Apply on create/update:

1. `goal_name` is required.
2. `goal_name` length: 1 to 80 characters after trimming whitespace.
3. `target_amount` is required and numeric.
4. `target_amount` must be greater than 0.
5. `target_amount` precision: max 2 decimal places.
6. If validation fails, keep user on dashboard and show a clear error message.

Suggested error messages:

1. `Goal name is required.`
2. `Goal name must be 1-80 characters.`
3. `Target amount must be a valid number.`
4. `Target amount must be greater than 0.`
5. `Target amount can have at most 2 decimal places.`

### 2.4 Progress Logic

Formula:

`progress_percentage = (net_savings / target_amount) * 100`

Rules:

1. If `net_savings < 0`, use `0` for progress input.
2. Clamp progress to `[0, 100]`.
3. Round for display to whole percent.

Exact color thresholds:

1. Red: `0 <= progress <= 50`
2. Green: `50 < progress < 100`
3. Gold: `progress == 100`

## 3. Dashboard Rendering (Jinja)

Place Goal section immediately above existing financial metric cards.

### 3.1 Active Goal Panel

When there is an active goal, show:

1. Goal label, e.g. `Goal: New Laptop - R10,000`.
2. Current net savings and target amount (formatted currency).
3. Progress text, e.g. `Progress: 75%`.
4. Progress bar using threshold colors above.
5. Update form with `goal_name` and `target_amount`.
6. `Save Goal` button.
7. `Complete Goal` button.
8. `Cancel Goal` button.

### 3.2 No Active Goal State

When no active goal exists, show:

1. Heading text: `Set a goal`.
2. Goal create form (`goal_name`, `target_amount`, submit button).
3. No progress bar is shown.

### 3.3 Past Goals Section

Show list of `completed` and `cancelled` goals with:

1. Goal name.
2. Target amount.
3. Final net savings.
4. Status.
5. End date (from `ended_at`).

If no past goals exist, show exact empty text:

`No Past Goals found`

## 4. Flask/Jinja Interaction Contract

Because this app is Jinja-based, use server-rendered form submissions and redirect back to dashboard.

Required behaviors:

1. Create/update active goal from form submission.
2. Complete active goal from button submission.
3. Cancel active goal from button submission.
4. After each action, persist to DB and re-render dashboard state on next GET.

Implementation detail is flexible:

1. Use separate POST endpoints or one POST endpoint with action field.
2. Preserve existing API endpoints and existing response formats.

## 5. UI Overhaul: 80s Financial Terminal

Redesign dashboard styling to look like an 80s high-stakes financial terminal.

### 5.1 Visual Direction

1. Background: dark obsidian tones.
2. Accent colors: Terminal Green `#00FF00` and Gold `#FFD700`.
3. Typography: monospaced family (`Courier New`, `Fira Code`, or equivalent fallback chain).
4. Cards/panels: thick solid borders instead of soft shadows.
5. Add CRT scanline overlay effect.
6. Badges and category badges should resemble glowing LED indicators.

### 5.2 Layout and Responsiveness

1. Goal block must be visually distinct from metric cards.
2. Progress bar and forms must be usable on desktop and mobile widths.
3. Keep contrast high enough for readability.

## 6. Data Reset and Migration Rule

Destructive reset is allowed for this tutorial task.

1. It is acceptable to delete the existing SQLite DB file.
2. Recreate and seed DB with:

`python -m data.seed`

## 7. Acceptance Criteria (Agent Must Meet)

1. User can create one active goal and see it on dashboard.
2. User can update active goal and it overwrites in place.
3. Goal never auto-completes based on progress alone.
4. User can click Complete Goal and status changes to completed.
5. User can click Cancel Goal and status changes to cancelled.
6. On completed/cancelled transitions, `final_net_savings` and `ended_at` are stored.
7. Progress is clamped 0 to 100 and color thresholds match exact rules.
8. Empty-state text appears exactly:
   1. No active goal: `Set a goal`
   2. No history: `No Past Goals found`
9. Currency values are formatted using existing app logic.
10. Dashboard visual style reflects 80s terminal direction.
11. Existing app functionality and tests remain functional or are updated if intentionally changed.
12. Tests pass and code is formatted with black. (Run `black .` from project root., and `pytest -v` to run tests.)
