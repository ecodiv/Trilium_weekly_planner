# Task Planner for TriliumNext

A weekly planner that surfaces tasks written inline in your notes. Type `TODO buy milk` anywhere in a daily note or project page; it appears in the planner. Schedule it by dragging into a day column. Mark it done and the source line gets greyed out in place.

![Overview: the planner with Backlog and seven day columns. It is full-width on the desktop. On mobile (window narrower than 700px), all columns are fixed-width and the board scrolls.](./screenshots/overview.png)

The tool is inspired by the [weekly planner](https://github.com/orgs/TriliumNext/discussions/9676) tool by [ricolandia](https://github.com/ricolandia), which in turn was inspired on the [Task-hub tool](https://github.com/ZangXincz/TriliumNext-Task-Hub) by [ZangXincz](https://github.com/ZangXincz). To tool was created making extensively use of AI. It was tested on Trilium version 0.102.0 and 0.103.0.

## Daily flow

One way to use e weekly planner:

1. **Capture as you go.** Type `TODO …` while you're in a meeting, project doc, or daily note. Don't think about where it goes — the planner will find it.
2. **Use `@today`, `@tomorrow`, or `@fri`** for things with a date in mind. Skip dates for everything else.
3. **Open the planner once a day.** Drag Backlog items into days based on what you want to actually do.
4. **Through the day**, mark items done by clicking the ✓ on each card. Greyed source lines tell you what you've shipped.
5. **Click a card** when you want to actually work on the task — the side panel opens the source note alongside the planner, so you can update it in context.
6. **End of week:** hit `↺` if the current week's plan was speculative and you want a clean slate. Unplanned tasks stay in Backlog.


## Setup

itself is a **Render Note** which runs inside Trilium as a regular note view. The state and configuration both are kept in a small JSON file inside a note you create once. The underlying JSX code is kept in a JSX file.

To setup the weekly planner, download the WeeklyPlanner.zip and import it into Trilium. The imported notes behave like a custom plugin. This imports three files:

1. planner.jsx: the code doing the magic.
2.  planner_data.json: the state note. Open this note, go to the attributes and add the label `#plannerdata`.
3.  planner_manual.md: this manual

For better organization, you may want to import everything inside a parent note such as “Tools”, “Plugins”, or “Addons”.

Next, create a note of type Render anywhere in your tree and add a relation `~renderNote` pointing to the JSX note you imported in step 1b. Open the note to run the planner

## Configuration
The weekly planner will use the `planner_data.json` note store which tasks are scheduled to which days, the order of cards within a day, the backlog width, and your active filters. 

You don't normally need to look at it, except when to change some of the options available. You can find these options under the `CONSTANTS` header in the json file. If you change any of these, make sure to refresh the planner (`SHIFT +CNTRL + R`)

**#backlogWidth=<pixels>**: Default width of the Backlog column on desktop. Example: `#backlogWidth=320`. Range 150–600. Without this label, the default is 260px. The user can interactively set the width by dragging the right edge of the Backlog column; the dragged value is saved into `#plannerdata` JSON and takes precedence over the label until cleared.

**#weekplanner_todo=<color>**: Override the TODO chip color. Accepts any CSS color: `#ed7a2a`, `red`, `rgb(120,60,200)`, `hsl(20 80% 60%)`. Without this label, the default orange is used.

**#weekplanner_idea=<color>**: Override the IDEA chip color. Default blue.

**#weekplanner_check=<color>**: Override the CHECK chip color. Default green.

**#weekplanner_toread=<color>**: Override the TOREAD chip color. Default purple.

If something goes wrong and the file gets corrupted, the [Recovery](#recovery) section shows how to edit or reset the JSON.

## How tasks work

The planner scans every text note in your tree for lines that start with one of four prefixes:

| Prefix | Meaning | Default color |
|---|---|---|
| `TODO` | Actionable task | orange |
| `IDEA` | Captured thought | blue |
| `CHECK` | To verify or review | green |
| `TOREAD` | Reading queue | purple |

Prefixes are case-sensitive and must be followed by a space. They must appear at the start of a paragraph, list item, or after a `<br>`. Anything else (e.g. `My TODO list:` in prose) is ignored.

### The day card

![Card with ✓ button visible on hover, kind chip, date suffix and tag pill](./screenshots/card.png)

Each task is rendered as a small card showing: the kind chip in its colour, the task text, the `@date` suffix (if any) in light grey, any `#tags` as grey pills, and the source note title below.

The ✓ button in the top-right corner is the "mark done" action. On desktop it appears on hover; on touch devices it's always visible.

### Marking done

Clicking the ✓ removes the task from the planner. The line stays in your source note, greyed out, as a record of completion. No data is lost. This also means there is no automatic cleanup. Delete them manually when you want a tidy source.

### Opening the source note

Clicking anywhere else on the card (not the ✓) opens the source note. The default opens it in a **side panel** alongside your current view. To open it as a new tab, you need to `Ctrl-click` / `Cmd-click`. Alternatively, you can use the `Middle-click` to open the note with the task in a new tab. 

## Scheduling

### By drag (desktop)

Drag a card between Backlog and any day column. A blue line shows where the card will land. Drop position is preserved within a day. I.e., the planner remembers card order per day, not just which day.

### By tap (mobile)

Tap a card to open the source note in the side panel. To schedule via tap on mobile, the easiest path right now is to drag. Though touch drag in the kanban can be fiddly on small screens. The `@date` suffix below is usually an easier way on mobile.

### By @date suffix

Append `@date` anywhere in the task text to auto-schedule:

```
TODO call the dentist @tomorrow
IDEA new pricing model @fri
CHECK Q3 invoices @2026-05-20
TOREAD Hofstadter essay @sat
```

Recognised tokens: `@today`, `@tomorrow`, `@mon` through `@sun`, and any ISO date `@YYYY-MM-DD`. Unrecognised tokens stay in the text and don't schedule.

### Backlog

Tasks without a planned date live in the Backlog column. Drag from a day back to Backlog to unschedule.

## Quick capture

The input bar above the board appends a new task to **today's daily note**, creating that note if it doesn't exist. By default the kind is TODO, but if you type a known prefix it's respected:

```
TODO send invoice           TODO assumed
TOREAD Hofstadter @sat      purple card, lands on Saturday
IDEA pricing model #work    blue card, tagged #work
```

Press Enter or click the `add` button to save the card. The board reloads automatically; `@date` suffixes are applied immediately.

## Tags

Add `#tag-name` anywhere in a task text to tag it:

```
TODO call mom #personal
TODO deploy staging #work #urgent
```

Tags display as small grey pills on the card and feed the filter dropdown. Tag rules: must start with a letter, can include letters/digits/underscores/hyphens. Tags stay in the source note and don't move.

## Filtering

![Filter dropdown showing kind checkboxes and tag list](./screenshots/filter.png)

Click the **Filter** button (top-right of the planner header). The dropdown shows:

- **Kinds** — checkboxes for each prefix type that has at least one task. Toggle to hide/show.
- **Tags** — checkboxes for every tag found across all tasks. Multiple tag selections are combined with **AND** (a task must have *all* selected tags to show).

If no kinds are selected, all kinds show. If no tags are selected, tag filtering is off. The active filter count appears as a small blue badge on the Filter button. **Clear filters** resets both.

Filters persist across reloads (stored in the `#plannerdata` JSON).

## Header controls

Left to right:

| Control | Action |
|---|---|
| **Planner** | Title |
| **‹** | Previous week |
| **(date range)** | Currently shown week |
| **›** | Next week |
| **today** | (only visible if off the current week) Jump to current week |
| **N/M planned** | N tasks scheduled this week out of M total |
| **Filter** | Open filter dropdown |
| **↺** | Clear *all* planning for the currently visible week (asks to confirm) |
| **⟳** | Re-scan all notes for tasks |

The reload button is useful after you've edited tasks in source notes — the planner doesn't watch for changes in real time.

## State persistence

The `#plannerdata` note's content is a JSON document storing:

- Which task is scheduled to which day (`taskId → ISO-date`)
- Within-day ordering (`_order: { ISO-date: [taskIds...] }`)
- Saved backlog width (`_backlogWidth: pixels`)
- Saved filters (`_filters: { kinds, tags }`)

A populated file looks roughly like this:

```json
{
  "abc123::TODO::call_the_dentist": "2026-05-15",
  "abc123::TODO::buy_milk": "2026-05-15",
  "def456::IDEA::new_pricing_model": "2026-05-17",
  "_order": {
    "2026-05-15": [
      "abc123::TODO::buy_milk",
      "abc123::TODO::call_the_dentist"
    ]
  },
  "_backlogWidth": 320,
  "_filters": { "kinds": [], "tags": ["work"] }
}
```

Keys that start with `_` are planner metadata. All other keys are task IDs mapping to ISO dates. The file is written automatically every time you drag a card, mark a task done, change filters, or resize the backlog. So you don't normally need to edit it. You can if you want, though: see [Recovery](#recovery) for safe ways to do that.

> [!IMPORTANT]Tasks are identified by a hash derived from `noteId`, kind, and the first 48 characters of text. Editing a task's text invalidates its ID, so its scheduled day is forgotten. If you need to edit a task without losing its day, drag the card to the same column afterwards — the new ID will be re-scheduled correctly.

## Recovery

If the planner won't load, or shows odd behaviour after a Trilium upgrade, the cause is almost always something in `#plannerdata`. From least to most destructive:

### Symptom: "Initialization error" or planner stuck on Loading…

Open the `#plannerdata` note and look at its content. If it's not valid JSON (you'll see the error in the planner's red error banner), the most common causes are:

- Trailing commas, smart quotes, or missing brackets if you hand-edited
- An aborted save (rare, but possible if Trilium crashed mid-write)

**Fix:** replace the entire content with a minimal valid state and reload:

```json
{}
```

That's it — an empty object. The planner will start with no scheduled tasks, default backlog width, no filters. All your source notes (and the tasks inside them) are untouched. You just lose the day-assignments and ordering.

### Symptom: planner loads but tasks aren't in the right days

Could be timezone-related (the planner uses local dates) or could be that some task IDs have changed (you edited a task's first 48 characters). Either way, the source notes are still correct; only the planner's mapping is off.

**Fix:** drag the affected cards to where they should be. The new state saves immediately.

If many cards are off — for example, you imported notes or migrated from another planner version — you can wipe just the scheduling portion of the state. Edit `#plannerdata` content to remove everything except metadata:

```json
{
  "_backlogWidth": 320,
  "_filters": { "kinds": [], "tags": [] }
}
```

All tasks become unscheduled (Backlog). Your configuration is preserved.

### Symptom: color override label has no effect

Three things to check:
1. The label name is exactly `weekplanner_todo` (etc.): lowercase with underscore
2. The value parses as a CSS color (try `#ff0000` first; if that works your color string was the problem)
3. You reloaded the planner (`⟳` button or F5) after adding the label

### Ultimate option

If something is deeply wrong, you can delete the `#plannerdata` note entirely and create a fresh one with just the label and an empty body. The planner will treat it as a first-time setup. No source-note data is at risk — `#plannerdata` only holds planner state, never task content.

## Limitations

- **Task IDs aren't permanent.** Editing the first 48 characters of a task's text in its source note changes its ID; its planned day mapping is lost. Drag it back where you want it.
- **DONE-marked items accumulate.** The greyed lines stay in source notes. There's no automatic cleanup. Delete them manually when you want a tidy source.
- **No real-time sync.** Edit tasks in source notes, hit `⟳` to reload the planner.
- **Mobile drag is awkward.** Touch drag works but isn't great on small screens. Prefer the `@date` suffix on mobile.

