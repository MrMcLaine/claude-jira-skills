# R calibration — local knowledge

Read by `r-assess` before every assessment; edited by it per the protocol in `../SKILL.md`.
This file is the part of the skill that learns. `references/framework.md` is the part that doesn't.

Everything below marked `seed` came from the framework's generic examples, not from our data —
replace with real anchors as precedents accumulate.

## Axis anchors

Cite as `A-<axis>-<n>`. An anchor is a reusable mapping "this kind of thing in our system → this
axis value", with its source (seed | P-NNN).

### Blast Radius

- `A-b-1` (seed) High: anything all clients see — public API contract, money/billing math, mass messaging.
- `A-b-2` (seed) Medium: one platform / one tenant / one segment of clients.
- `A-b-3` (seed) Low: internal tooling, team-only scripts, admin UI nobody outside sees.

### Reversibility

- `A-v-1` (seed) High: revert or flag-off fully undoes it.
- `A-v-2` (seed) Medium: undo exists but costs work — backfill, reverse migration, manual cleanup.
- `A-v-3` (seed) Low: data deleted, money moved, message delivered — no undo exists.

### Detectability

- `A-d-1` (seed) High: a test or alert fires within an hour of the mistake.
- `A-d-2` (seed) Medium: dashboards or client complaints within days.
- `A-d-3` (seed) Low: found by accident weeks later, or never.

### Bought detect (accumulates — inherited by every future task in the area)

_None recorded yet. Format: `A-d-bought-1: <area> — <alert/invariant>, proof shown <date>, source P-NNN`._

## Precedents

Human overrides of skill proposals — the primary learning signal. Newest last. Format:

```
### P-001 (YYYY-MM-DD) <task> — <one-line title>
Changed: <field> <from> → <to>. Why: <their reason>.
Rule: <one transferable sentence a future assessment applies>.
```

_None yet._

## Disputed ✳ cells — local case law

Six cells where the axes contradict; the framework says calibrate these first. Log every real case
that lands here; a convention emerges from cases, not from argument.

### Cell 3 (B Low · V High · D Low → R2+) — e.g. local script nobody cross-checks
_no cases yet_

### Cell 6 (B Low · V Med · D Low → R2+)
_no cases yet_

### Cell 7 (B Low · V Low · D High → R2) — irreversible but tiny radius
_no cases yet_

### Cell 8 (B Low · V Low · D Med → R2)
_no cases yet_

### Cell 9 (B Low · V Low · D Low → R2+) — small, irreversible, silent
_no cases yet_

### Cell 19 (B High · V High · D High → R2) — flag rolled out to everyone
_no cases yet_

## Limits in force

Framework starting guess — a guess, not a recommendation; recalibrate from journal evidence only.

- Per person per week: **1×R3, 3×R2** (concurrent, not per-approve-date).
- The reviewer spends their limit too.
- A task without R is not planned.

## Calibrate-run log

_One line per calibrate run: date, entries covered, what was promoted/retired._
