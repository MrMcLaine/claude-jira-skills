---
name: r-assess
description: Assess a task's R responsibility rating (R0–R3, optional +) from Blast Radius, Reversibility, Detectability; propose buy-downs (alert, flag, staged rollout) BEFORE fixing the final R; log every assessment to a local calibration journal and learn from human overrides. Use when asked to "assess R", "оцени R", "выстави R", "який R у задачі", "какой тут R", "R-оценка", "responsibility rating", when planning/grooming tasks that need an R, or before jira-create fills the R lozenge. Also handles "R calibrate" / "калибровка R" consolidation runs.
argument-hint: [Jira key, ticket text, plan or diff — e.g. "OT-1365" or "новая таблица attribution jobs"; or "calibrate"]
---

# Assess R — responsibility rating with local calibration

R (`R0`–`R3`, optional `+`) measures what the person clicking **Approve** risks — not how much
work the task holds. This skill derives R from three axes, proposes **buy-downs before fixing the
final number**, and — the part that makes it grow — **records every assessment and every human
override locally**, so each run is calibrated by the previous ones.

Full framework (axis definitions, all 27 combinations, greenfield, system design, measurement):
`references/framework.md`. Read it when an assessment is contested, hits a ✳ cell, is greenfield /
system-design / connect-task shaped, or when running calibrate mode.

## Assets

- `references/framework.md` — the full framework reference (stable; do not edit during assess).
- `calibration/CALIBRATION.md` — **local knowledge**: team axis anchors, precedents from overrides,
  disputed-cell case law, limits in force. Read before EVERY assessment; edit per the protocol below.
- `calibration/journal.jsonl` — append-only assessment log. Never rewrite existing lines.

## Hard rules (from the framework — not negotiable)

1. **No axis judged → no R.** If any axis cannot be assessed, do not emit a number; name the
   missing judgment and ask for it. No number — no signature.
2. **Epics never get R.** A container spends no attention. Offer a roll-up over children instead.
3. **Judge by purpose, not today's environment.** Unconnected greenfield code is rated by whose
   mistake it becomes when live. Only a prototype destined to be thrown away is R0 by fate.
4. **`+` requires a named reason** (goes into the ticket). Matrix-`+` (Detectability = Low) is
   removable only by buying detect.
5. **Final R is set AFTER the buy-down pass**, never before. Detect is proposed by default.
6. **A purchase counts only with proof** (the alert demonstrably fired, the flag demonstrably
   kills the path). Until proof, treat the task at the pre-purchase level.
7. **R3+ is a signal, not a level**: recommend buying reversibility and returning with a smaller
   number, not deepening review further.

## Mode: assess (default)

### 1. Load local knowledge

Read `calibration/CALIBRATION.md`. Note applicable axis anchors (by id `A-…`) and precedents
(`P-…`) — you will cite them in the output. If the file has grown stale hints (e.g. a "propose
calibrate run" note), mention it at the end.

### 2. Identify the subject

From `$ARGUMENTS` or conversation: a Jira key (fetch via Atlassian MCP if available), ticket text,
plan, or diff. Classify `kind`:

- `delivery` — normal code/config change (default)
- `system-design` — a design ticket: axes read against the decision (who inherits it / cost to
  revisit / when wrongness surfaces) — see framework.md § System Design
- `connect` — enabling/rollout task: R covers switch mechanics only (routing, flag, gradualness,
  rollback); the accumulated R was spent by the layer tasks
- `prototype` — will be thrown away → R0 by fate, say so and stop
- `epic` — hard rule 2: no R, offer roll-up

### 3. Judge the three axes

For each axis give a value **High / Medium / Low** and a one-line why, citing anchors/precedents
where they apply:

| Axis          | Question                  | High                                | Medium                       | Low                             |
| ------------- | ------------------------- | ----------------------------------- | ---------------------------- | ------------------------------- |
| Blast Radius  | who sees it if wrong      | all clients, money, public contract | some clients, one tenant     | only me / the team              |
| Reversibility | what undoes it            | revert, flag off                    | backfill, reverse migration  | data deleted, money gone, email delivered |
| Detectability | what fires, when          | test/alert within an hour           | metrics/complaints in days   | by accident in weeks, or never  |

Polarity: Blast Radius High is bad; the other two High is good. If the human's stated judgment
differs from yours, theirs wins — but that difference is a **precedent** (step 7).

### 4. Compute base R

- `E = worst(Reversibility, Detectability)`
- Matrix (Radius × E):

  |            | E High | E Med | E Low |
  | ---------- | ------ | ----- | ----- |
  | **B Low**  | R0     | R1    | R2 ✳  |
  | **B Med**  | R1     | R2    | R2    |
  | **B High** | R2 ✳   | R2    | R3    |

- Auto-`+` if Detectability = Low. Add human `+` only with a named reason (unfamiliar area, code
  DRI doesn't own, first of its kind, external dependency, deadline).
- Note the cheatsheet row number (1–27) and whether it is a ✳ cell — ✳ assessments deserve a
  sentence more of justification and land in the disputed-cell log at step 7.

### 5. Buy-down pass (before finalizing)

Propose purchases, **detect first and by default**:

| Axis          | Typical purchase                                                        | Paid with     |
| ------------- | ----------------------------------------------------------------------- | ------------- |
| Detectability | alert, invariant assert, reconciliation, metric shipped BEFORE the change | SP (few)      |
| Reversibility | feature flag, dry-run, hold queue, cancellation window                   | design        |
| Blast Radius  | 1% traffic, one tenant, employees first                                  | calendar time (product decision) |

Rules: detect drops the **level** only when Reversibility ≠ Low (otherwise it removes only the
`+`); detect purchases accumulate area-wide (first task pays, the rest inherit — record inherited
detect as an anchor); each purchase must name its **proof**.

### 6. Output — fixed block

```
R: <final> (base <base>, cell <n><, ✳ if so>: B <v> · V <v> · D <v> → E <v>)
+ reasons: <named, or "—">
Applied local knowledge: <A-…/P-… ids, or "none — first case of its shape">
DRI must: <depth line from the table below>
Reviewer must: <depth line>
Buy-down:
  - [detect, default] <what> — proof: <what demonstrates it> → <resulting R>
  - [reversibility] <what> → <resulting R>          (only if applicable)
  - [radius] <what> → <resulting R>                 (only if it's worth raising as a product call)
Conditional: with <purchase> proven → <lower R>; until then treat as <higher R>.
Limits: <DRI> holds <n>×R3 / <n>×R2 this week (limit <l3>/<l2> from CALIBRATION.md) — <OK | OVER>.
```

Depth table:

| R  | DRI                                            | Reviewer                                      |
| -- | ---------------------------------------------- | --------------------------------------------- |
| R0 | green tests are enough                         | diff scan, AI review                          |
| R1 | read the diff, check edges                     | AI review + scan, questions on edges          |
| R2 | take apart the solution; verify tests test meaning | read every line; AI review is a first pass |
| R3 | reproduce manually / dry-run; state the invariant out loud | read in full; approve only after rollback talked through |

`+` = one row lower. For the limits line, count this ISO week's journal entries per DRI at final
R ≥ R2 (skip silently if the journal has no `dri` data yet).

### 7. Record — ALWAYS, this is the self-improvement loop

**a. Journal.** Append ONE line to `calibration/journal.jsonl`:

```json
{"ts":"<YYYY-MM-DD>","task":"<key or short slug>","title":"<one line>","kind":"delivery","dri":"<who or null>","axes":{"b":"Med","v":"Med","d":"Low"},"e":"Low","base_r":"R2+","cell":15,"disputed":false,"purchases":[{"axis":"d","what":"alert X","proof":"fires on synthetic event","status":"proposed"}],"final_r":"R2+","plus_reasons":["matrix: D=Low"],"overrides":[],"precedent":null}
```

`purchases[].status`: `proposed` | `bought` (proof shown) | `declined`. `overrides`: filled only
when the human changed something (see b).

**b. Precedents — the gold.** If the human corrected ANY part of your proposal (an axis value, the
final R, a `+`, a purchase verdict), immediately:

1. Append the override to the journal line: `{"field":"axes.b","from":"High","to":"Med","why":"<their reason>"}`.
2. Add a precedent to `CALIBRATION.md` § Precedents: next id `P-NNN`, date, task, what changed,
   and **one extracted transferable rule** (a sentence a future assessment can apply — not a
   restatement of this case). Set `"precedent":"P-NNN"` in the journal line.
3. If the rule names a concrete system area ("X is Medium radius because single-tenant"), also
   fold it into § Axis anchors — new anchor or a refinement of an existing one, marked with the
   precedent id as source.

**c. Disputed cells.** If the assessment landed on a ✳ cell, add the case one-liner under that
cell in `CALIBRATION.md` § Disputed cells (id = journal ts+task).

Never edit `references/framework.md` during assess, and never rewrite journal history — the
journal is append-only; CALIBRATION.md is the only file that evolves per-assessment.

### 8. Ticket handoff (when a Jira ticket is being created/updated)

Provide the jira-create placeholders — the R block lives in the TL;DR panel next to Complexity:

- `{{R_LEVEL}}`: `R0`…`R3+` · `{{R_COLOR}}`: R0 → `green`, R1 → `yellow`, R2/R2+/R3/R3+ → `red`
- `{{R_AXES}}`: `B Med · V Med · D Low → E Low` (append ` · cell 15` if useful)
- `{{R_NOTE_1}}` bullets: each `+` reason (**mandatory** when `+` is present — a `+` without a
  named reason in the ticket is invalid), and each pending purchase as
  `buys: <what> (proof: <…>) → <R>`
- Jira **label** in `additional_fields`: `r0` | `r1` | `r2` | `r2-plus` | `r3` | `r3-plus` —
  this is what makes R queryable (In-Review time per R, reviewer concentration).

A task without R is not planned — if jira-create is about to create a plannable ticket with no R,
run this skill first. Epics: drop the R block entirely.

## Mode: calibrate (on "calibrate" / "калибровка R" / journal grown by ~20 entries)

A consolidation pass over local knowledge — the skill improving itself deliberately:

1. Read all of `calibration/journal.jsonl` + `CALIBRATION.md`.
2. Report: entries per final R (sanity: R0 above ~10% ⇒ radius understated — say so); ✳-cell hit
   counts; override rate per axis (which axis do humans correct most — that axis's anchors are
   weakest); purchases proposed vs actually bought (proof shown) vs declined.
3. Propose edits to `CALIBRATION.md`, then apply the confirmed ones:
   - promote a rule seen in ≥2 precedents into an axis anchor; mark superseded precedents;
   - flag anchors contradicted by newer precedents — never keep both silently;
   - update § Disputed cells with any emerging local convention ("in our team cell 7 tasks get a
     mandatory dry-run instead of R2 review depth");
   - propose limit changes only from evidence (e.g. journal shows weeks with 2×R3 on one DRI and
     visible In-Review pile-ups) — limits are the team's call, keep the current numbers until
     confirmed.
4. Never delete journal lines; never edit `references/framework.md` — if calibration reveals the
   framework itself should change, say so explicitly and let the human change it.

## Notes

- Org Jira: `ofms.atlassian.net`; prefixes `OT`, `COR`, `AI`. Fetch tickets via
  `mcp__atlassian__getJiraIssue` when a key is given.
- Language: assess in the conversation's language; write journal/CALIBRATION entries in English
  (greppable, consistent), quoting the human's wording where it carries the nuance.
- If this skill directory is installed as a **copy** (not a symlink to the claude-jira-skills
  repo), calibration writes land in the copy and diverge — recommend the symlink install from the
  repo README once per session if you notice it.
