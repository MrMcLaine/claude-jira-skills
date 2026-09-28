---
name: jira-create
description: Create a beautiful, dashboard-style JIRA issue (Task, Bug, Story, Epic, Subtask) using rich ADF via the official Atlassian MCP. Use when asked to "create a JIRA ticket/issue", "open a bug in JIRA", "file a story", or "make a JIRA epic/subtask".
argument-hint: [what to create, e.g. "Bug: login button unresponsive on mobile in PROJ"]
---

# Create a Rich JIRA Issue

Create a Jira issue with a **dashboard-style description** — colored panels, status lozenges,
collapsible sections, interactive checklists, and tables — instead of flat text. Uses the official
Atlassian Remote MCP (`mcp__atlassian__*`). This skill covers **issue creation only** — not search,
transitions, or agile operations.

## Tool names (official Atlassian Remote MCP)

Use these exactly — community-server names differ:

- `mcp__atlassian__getAccessibleAtlassianResources` — resolve `cloudId` (required by every call)
- `mcp__atlassian__getVisibleJiraProjects` — list/verify projects (pass `action: "create"`)
- `mcp__atlassian__getJiraProjectIssueTypesMetadata` — valid issue types for a project
- `mcp__atlassian__getJiraIssueTypeMetaWithFields` — required/custom fields for an issue type
- `mcp__atlassian__lookupJiraAccountId` — resolve assignee email/name → account ID
- `mcp__atlassian__createJiraIssue` — create the issue

## Assets this skill loads on demand

- `templates/task.adf.json`, `templates/bug.adf.json`, `templates/story.adf.json` — fill-in ADF docs.
- `../../shared/adf-cheatsheet.md` — the full ADF node reference. Read it when building/extending a
  description or when a render looks wrong.

## Workflow

### 1. Resolve `cloudId` (once per session)

`cloudId` is **required** on every Jira call.

- Call `getAccessibleAtlassianResources` (no args) and use the returned site id. A site URL
  (e.g. `your-org.atlassian.net`) is also accepted as `cloudId`.
- Reuse the same `cloudId` for all subsequent calls in the session.

### 2. Gather requirements — NEVER ASSUME the project

Collect, asking the user for anything missing:

- **Project key** (e.g. `COR`, `AI`, `OT`) — never guess; verify it exists.
- **Issue type** — Task, Bug, Story, Epic, or Subtask.
- **Summary** — concise title.
- The content for the chosen template's sections (objective, acceptance criteria, steps, etc.).
- Optional: **priority**, **assignee**, **labels**, **parent** (for Subtask), **target date**.

If the user gave a one-line request, infer type + summary from it, but still confirm the project key
if it is not obvious from context.

### 3. Validate project and issue type

- Verify the project: `getVisibleJiraProjects` with `action: "create"` (filter via `searchString`).
- Confirm the issue type is available: `getJiraProjectIssueTypesMetadata`.
- Only on "field required" errors or to find a custom field ID, call
  `getJiraIssueTypeMetaWithFields`.

### 4. Resolve assignee (only if requested)

`createJiraIssue` takes `assignee_account_id`, not an email. If the user named an assignee, call
`lookupJiraAccountId` with the email or display name and pass the resolved id.

### 5. Build the ADF description

Pick the template matching the issue type:

| Issue type     | Template               |
| -------------- | ---------------------- |
| Task / Subtask | `templates/task.adf.json` |
| Bug            | `templates/bug.adf.json`  |
| Story / Epic   | `templates/story.adf.json` |

Read the template, then **replace every `{{PLACEHOLDER}}`** with real content. Rules:

- **Drop sections the user has nothing for** — delete the whole node rather than leaving `{{...}}`
  or an empty string. A lean ticket beats a skeletal one.
- **Add rows/items freely** — duplicate a `taskItem` / `listItem` / `tableRow` for more entries,
  giving each a **unique `localId`** (`ac1`, `ac2`, `ac3`, …). Duplicate ids render incorrectly.
- **Describe the plan, not the progress.** A description is **static** — it never auto-updates, so
  anything reflecting live state or an outcome will rot and require manual edits. Include only what
  is **known and stable at creation time**. EXCLUDE:
  - **Status** — Jira shows the workflow status natively.
  - **Assignee / owner** — native Assignee field.
  - **Shipped version, delivered PR/commit links, release notes** — the development panel and
    fixVersion track these.
  - **Test pass/fail results** — a Test *Plan* (scenario + expected) is the spec; results are an
    outcome, so omit a "Result" column.
  - Leave **acceptance criteria and Definition of Done unchecked** (`state: "TODO"`) — they are
    goals, not claims of completion.
- **Lozenges are for stable facts NOT owned by a Jira field** — e.g. complexity, estimate,
  severity, environment. Never use a lozenge for Status, Priority, Components, or anything
  in the "never duplicate native Jira fields" list below.
- **Use the fixed color palette — never pick colors yourself.** Colors carry meaning; map them
  deterministically:

  | Color    | Meaning            | Used for                                       |
  | -------- | ------------------ | ---------------------------------------------- |
  | `blue`   | Info               | TL;DR panel                                    |
  | `green`  | Ready / Fact / Low | Impact panel, Low complexity                   |
  | `yellow` | Warning / Medium   | Out of Scope panel, Medium complexity          |
  | `red`    | Risk / High        | High complexity                                |
  | `purple` | Metadata           | Estimate                                       |
  | `neutral`| Neutral            | Environment and other non-graded facts         |

  Level → color is fixed: **Low → green · Medium → yellow · High → red.**
- **Date pills** (`{{*_TIMESTAMP_MS}}`): the value is **epoch milliseconds as a string**. Convert
  the user's date first — e.g. `2026-06-11` → `"1781481600000"`. Compute it with a quick shell call
  if needed: `date -d 2026-06-11 +%s000` (GNU) or `date -j -f %Y-%m-%d 2026-06-11 +%s000` (macOS).
  If no date is given, delete the date line.
- Need a node the template lacks (extra panel, mention, link)? See `../../shared/adf-cheatsheet.md`.

### 5a. Canonical section structure

Tickets of the same issue type MUST share the same skeleton and section order. The `task` template
defines the canonical order — keep it; drop sections that don't apply, never reorder or rename them:

0. **Specification notice** (top note panel) — the fixed disclaimer (see content rules). Always
   the first node of every ticket, verbatim.
1. **TL;DR banner** (blue info panel) — one sentence on what the change is, plus a **Complexity**
   lozenge and its **reason** bullets (grooming signal; see content rules), plus an **R** lozenge
   with its axes line and notes (responsibility rating; see content rules).
2. **🎯 Objective** — the goal, in one line.
3. **💡 Impact** — why now / what it unblocks or removes (e.g. "removes N calls …").
4. **📋 Context** — the current state and why it's insufficient.
5. **🔌 API Contract** — for endpoint work, a **collapsible** `expand` holding a code block:
   request, response shape, **error cases**, limits, and **ordering**. Hidden by default so the
   contract reads as a technical detail, not the headline.
6. **✅ Acceptance Criteria** — testable, unchecked **checklist** (`taskList`, not bullets).
7. **🔧 Technical Notes** — implementation guidance / gotchas (collapsible `expand`).
8. **🧪 Test Plan** — **table**: Scenario + Expected (no result column).
9. **⚠️ Risks** — **table**: Risk · Impact · Mitigation.
10. **📊 Observability** — **table**: Metric · Purpose.
11. **🚫 Out of Scope** — yellow warning panel: excluded work + where it's tracked.
12. **🏁 Definition of Done** — unchecked completion criteria.

### 5b. Content rules (what makes a good ticket)

- **Start with the specification notice (mandatory, verbatim).** The first node of every ticket is
  a note panel containing exactly:
  > **This description is a specification.**
  > Do not describe implementation unless it is already decided. Do not describe completed work.
  > Do not describe future outcomes. Describe only known requirements.

  This frames the whole ticket — honour it while filling every section.
- **Prefer consistency over creativity.** The same type of change should produce
  similarly-structured tickets. Reuse the canonical skeleton, section order, emoji, and lozenge
  palette every time — a reader should recognize the shape before reading a word. Do not invent new
  sections or reorder per ticket.
- **Always lead with a TL;DR** — a single sentence right after the notice, e.g.
  *"Batch endpoint to fetch full account profiles by explicit account_ids — max 200 per request."*
- **Acceptance Criteria must be testable.** Each line should be verifiable by a concrete check.
  - Bad: `Existing endpoint unaffected`
  - Good: `GET /api/v0/internal/accounts/:account_id returns the same response as before for existing callers`
- **Out of Scope is mandatory** — always state what is *not* included and where it's tracked, so the
  boundary is explicit. Keep "Done/in-scope" and "Out of scope" visually separate.
- **API Contract for any endpoint change** — put it in a **collapsible `expand`** (it's a technical
  detail, not the headline) holding a code block that shows: the request, the success response
  shape, **every error case** (status + body, e.g. `400 { "message": "account_ids exceeds limit" }`
  — most tickets wrongly describe only the happy path), the limits (min/max, invalid/unknown input),
  and the **ordering** of returned collections (request order vs DB order — never ambiguous).
- **Risks and Observability are expected** for non-trivial changes, and MUST be **tables**:
  - Risks → `Risk | Impact | Mitigation`.
  - Observability → `Metric | Purpose` (use real metric names, e.g. `accounts.batch.requests`).
- **Naming consistency** — pick one term for each concept and use it everywhere (e.g. always
  `account_ids`, never mix in `ids`). Mirror the exact field/param names from the code.
- **Complexity lozenge (required, and always justified).** Put a Complexity level in the TL;DR
  banner — a lozenge **Low `green` / Medium `yellow` / High `red`** — and **always follow it with a
  short `reason:` bullet list**. A bare "Complexity: Medium" is not acceptable. Example:
  > Complexity: Medium — reason: • new endpoint • new validation • no schema changes

  Rubric:
  - **Low** — additive endpoint, internal refactoring, config-only change.
  - **Medium** — schema changes, caching changes, new dependency, cross-service contract tweak.
  - **High** — data migrations, breaking API changes, auth/permission changes, irreversible ops.
- **R lozenge (responsibility rating — required on every plannable ticket).** Complexity says how
  much work; **R** (`R0`–`R3`, optional `+`) says what the approver risks. It is produced by the
  **`r-assess` skill** (`../r-assess/SKILL.md`) — if the task has no R yet, run it first; a task
  without R is not planned. **Epics never get R** — delete the R paragraph and its bullets.
  Filling the template placeholders:
  - `{{R_LEVEL}}` — `R0`…`R3+`; `{{R_COLOR}}` is fixed: **R0 → `green` · R1 → `yellow` ·
    R2/R2+/R3/R3+ → `red`**.
  - `{{R_AXES}}` — the derivation, e.g. `B Med · V Med · D Low → E Low`.
  - `{{R_NOTE_1}}` bullets — one per note: each `+` reason (**mandatory whenever the level carries
    a `+`** — a `+` without a named reason is invalid), and each pending buy-down as
    `buys: <what> (proof: <how it's demonstrated>) → <resulting R>`. No notes → delete the bulletList.
  - Also add the matching label to `additional_fields.labels`:
    `r0` | `r1` | `r2` | `r2-plus` | `r3` | `r3-plus` — this makes R queryable in JQL (review-time
    stats per R, reviewer concentration).
- **Never duplicate native Jira fields.** Information Jira already owns must NOT be repeated in the
  description — it goes stale and contradicts the source of truth. Do not put any of these in the
  body; set them as real fields instead:
  - **Status** (workflow), **Assignee**, **Priority**, **Labels**, **Components**, **Fix Versions**.
  - Use `additional_fields` for priority/labels/components/fixVersions, and `assignee_account_id`
    for the assignee — never a lozenge or a line of text in the description.

### 6. Create the issue

Call `createJiraIssue` with the filled ADF as `description` and `contentFormat: "adf"`. Top-level
params: `cloudId`, `projectKey`, `issueTypeName`, `summary`, `description`, `contentFormat`, and
`parent` (Subtasks) / `assignee_account_id` (if resolved).

Everything else (priority, labels, components, fixVersions, custom fields) goes in
**`additional_fields`** — the only place those are accepted:

```json
{
  "cloudId": "<cloudId>",
  "projectKey": "OT",
  "issueTypeName": "Bug",
  "summary": "Login button unresponsive on mobile",
  "contentFormat": "adf",
  "description": { "type": "doc", "version": 1, "content": [ /* filled template */ ] },
  "additional_fields": {
    "priority": { "name": "High" },
    "labels": ["mobile", "login"],
    "fixVersions": [{ "name": "1.1.0" }],
    "components": [{ "name": "API" }]
  }
}
```

### 7. Report back

Return the new issue key and a clickable URL: `https://<site>/browse/<KEY>`.

## `additional_fields` format reference

| Field          | Shape                     |
| -------------- | ------------------------- |
| priority       | `{ "name": "High" }`      |
| labels         | `["label1", "label2"]`    |
| components     | `[{ "name": "API" }]`     |
| fixVersions    | `[{ "name": "1.1.0" }]`   |
| custom select  | `{ "value": "Option A" }` |
| custom number  | `5`                       |
| custom text    | `"some value"`            |

Find custom field IDs (`customfield_XXXXX`) via `getJiraIssueTypeMetaWithFields`.

## Markdown fallback

If rich rendering isn't wanted (or a field rejects ADF), pass `contentFormat: "markdown"` with a
plain markdown `description`. You get headings, tables, code, and `- [ ]` checklists — but no
panels, lozenges, or expands.

## Troubleshooting

- **Placeholders showing literally** (`{{AC_2}}`) — an unfilled token shipped; edit the issue and
  remove it, or delete unused nodes before creating.
- **Checklist / status renders blank or merged** — duplicate `localId`s. Make each unique.
- **Date shows as a number** — `timestamp` must be epoch **ms** as a string, not ISO or seconds.
- **"Project not found"** — keys are case-sensitive; list via `getVisibleJiraProjects`.
- **"Field required but not provided"** — call `getJiraIssueTypeMetaWithFields`; put the field in
  `additional_fields`.
- **"Invalid issue type"** — confirm via `getJiraProjectIssueTypesMetadata`.
- **Assignee fails** — resolve the account id with `lookupJiraAccountId` first.

## Notes

- This org's JIRA is at `ofms.atlassian.net`; recent ticket prefixes include `COR`, `AI`, `OT`.
- Dates/times use ISO 8601 from the user; convert to epoch ms only for ADF `date` nodes.
- For search/transition/sprint operations, use the relevant `mcp__atlassian__*` tools directly.
