# Gantt Creator

A single-file, browser-based Gantt chart tool with a text DSL for defining lanes and tasks. Designed for meal prep planning and similar scheduling problems where multiple resources (people, appliances) work in parallel.

Open `index.html` in any modern browser — no build step, no dependencies beyond a CDN link to Tailwind CSS.

## Features

- **Text-based DSL** for lanes and tasks — paste in, get a chart
- **Live bidirectional sync** — edits in the DSL update the chart; dragging blocks updates the DSL
- **Auto-scheduling** — unpinned tasks are placed automatically respecting dependencies and lane availability
- **Drag to reschedule** — 5-minute grid snap by default, hold Shift for 1-minute precision
- **Magnetic snapping** — tasks snap to neighboring task edges when close
- **Dependency arrows** — visualized with curves; violations highlighted in red
- **Overlap detection** — two active tasks on the same lane are flagged as violations
- **Per-lane passive tasks** — a task can occupy a lane without blocking it
- **Right-click lane reassignment** — move a task between sub-lanes of the same group
- **Import / Export** — save and load `.gantt` files
- **Zoom** — Ctrl+scroll to zoom in/out
- **Current time indicator** — optional "NOW" line

## DSL Reference

### Time Range

Set the visible time window with the Start and End time inputs. Times use `HH:MM` format (24-hour).

### Lanes

One lane per line. Lanes represent resources (people, equipment, etc.).

```
Name
Name: count
```

| Field | Description |
|-------|-------------|
| `Name` | Lane group name (e.g. `Person`, `Oven`) |
| `count` | Number of sub-lanes (optional, defaults to 1) |

**Examples:**

```
Person: 2       # Creates "Person 1" and "Person 2"
Oven            # Creates just "Oven"
Stove: 3        # Creates "Stove 1", "Stove 2", "Stove 3"
```

### Tasks

One task per line. Fields are pipe-separated:

```
Name | Duration | Lane(s) | Color | Dependencies | @Start
```

| Field | Required | Description |
|-------|----------|-------------|
| `Name` | Yes | Unique task name |
| `Duration` | Yes | Time length: `15m`, `1h`, `1h30m`, `90m` |
| `Lane(s)` | Yes | Which lanes the task occupies (see below) |
| `Color` | No | Hex color like `#3b82f6` (defaults to blue) |
| `Dependencies` | No | `after Task A, Task B` — tasks that must finish first |
| `@Start` | No | `@09:30` — pin to a specific start time |

Fields after Color (dependencies, start time) can appear in any order.

### Lane Specifiers

The lane field supports several formats, comma-separated:

| Syntax | Meaning |
|--------|---------|
| `Person` | Use any available sub-lane from the Person group |
| `Person*2` | Require 2 sub-lanes from the Person group simultaneously |
| `Person#1` | Pin to a specific sub-lane (Person 1) |
| `Person, Oven` | Occupy one Person sub-lane and the Oven |
| `Person(P)` | Passive on Person — tracked visually but doesn't block the lane |
| `Person#1(P)` | Passive on a specific sub-lane |

**Passive lanes**: Append `(P)` to any lane specifier. The task appears on that lane but won't prevent other tasks from being scheduled there. A task can be active on some lanes and passive on others.

### Examples

```
# Simple tasks
Chop vegetables | 15m | Person | #3b82f6
Boil water | 10m | Person | #ef4444

# Task with dependency
Cook pasta | 20m | Person | #f59e0b | after Boil water

# Pinned start time
Preheat oven | 10m | Oven | #a855f7 | @08:00

# Multi-lane task (needs 2 people and the oven)
Plate meal | 10m | Person*2, Oven | #10b981 | after Cook pasta

# Pinned to specific sub-lane
Prep station A | 15m | Person#1 | #3b82f6

# Passive task (background music doesn't block the person)
Music playing | 60m | Person(P) | #6b7280

# Mixed: active on Oven, passive on Person
Monitor bake | 45m | Oven, Person(P) | #a855f7
```

## Interaction

| Action | Effect |
|--------|--------|
| **Drag** a task block | Reschedule horizontally (5-min snap) |
| Hold **Shift** while dragging | 1-minute snap precision |
| **Right-click** a task | Move between sub-lanes of the same group |
| **Ctrl+scroll** | Zoom in/out |
| **Hover** a task | Show timing details and violations |

When dragging, tasks magnetically snap to the start/end edges of neighboring tasks on the same lane when closer than 5 minutes.

Changes made by dragging or reassigning are immediately reflected back into the DSL text.

## Render Button

The chart updates live when the DSL is fully pinned (every task has a `@start` time and all lanes are resolved). When the input is ambiguous (unpinned tasks that need auto-scheduling), a **Render** button appears. Clicking it runs the auto-scheduler, renders the chart, and writes back the resolved DSL.

## File Format (.gantt)

Export and import `.gantt` files via the Export/Import buttons. The format is plain text:

```
VERSION 1

START 09:00
END 14:00

LANES
Person: 2
Oven

TASKS
Chop vegetables | 15m | Person#1 | #3b82f6 | @09:00
Boil water | 10m | Person#2 | #ef4444 | @09:00
Cook pasta | 20m | Person#1 | #f59e0b | after Boil water | @09:10
Bake chicken | 45m | Oven | #a855f7 | @09:10
Plate meal | 10m | Person#1, Person#2, Oven | #10b981 | after Cook pasta, Bake chicken | @09:55
```

- `VERSION` — file format version (currently 1)
- `START` / `END` — time range in HH:MM
- `LANES` — section containing lane definitions
- `TASKS` — section containing task definitions

Exported files are always fully pinned (all times and lanes resolved), making the format idempotent: import → export produces identical output.
