# ADF Cheat Sheet

Atlassian Document Format (ADF) is the JSON document model Jira uses for rich content. Pass it to
`createJiraIssue` / `editJiraIssue` with `contentFormat: "adf"` and the `description` set to an ADF
**doc** object.

Every ADF document is:

```json
{ "type": "doc", "version": 1, "content": [ /* block nodes */ ] }
```

## Block nodes (top-level content)

### Heading

```json
{ "type": "heading", "attrs": { "level": 3 }, "content": [{ "type": "text", "text": "🎯 Objective" }] }
```

### Paragraph

```json
{ "type": "paragraph", "content": [{ "type": "text", "text": "Plain text." }] }
```

### Panel (colored callout)

`panelType`: `info` (blue) · `note` (purple) · `success` (green) · `warning` (yellow) · `error` (red).

```json
{ "type": "panel", "attrs": { "panelType": "warning" },
  "content": [{ "type": "paragraph", "content": [{ "type": "text", "text": "Depends on OT-1011." }] }] }
```

### Blockquote

```json
{ "type": "blockquote", "content": [{ "type": "paragraph", "content": [{ "type": "text", "text": "What done looks like." }] }] }
```

### Expand (collapsible section)

```json
{ "type": "expand", "attrs": { "title": "🔧 Technical notes" },
  "content": [{ "type": "paragraph", "content": [{ "type": "text", "text": "Hidden until clicked." }] }] }
```

### Code block

`language` is any Prism language id (`typescript`, `json`, `bash`, `sql`, …).

```json
{ "type": "codeBlock", "attrs": { "language": "typescript" },
  "content": [{ "type": "text", "text": "POST /accounts/batch // max 100 ids" }] }
```

### Task list (interactive checkboxes)

`state`: `TODO` or `DONE`. Every `taskList` and `taskItem` needs a unique `localId`.

```json
{ "type": "taskList", "attrs": { "localId": "ac" }, "content": [
  { "type": "taskItem", "attrs": { "localId": "ac1", "state": "TODO" },
    "content": [{ "type": "text", "text": "Endpoint returns 200" }] }
] }
```

### Decision list

`state` is always `DECIDED`. Needs unique `localId`s.

```json
{ "type": "decisionList", "attrs": { "localId": "dod" }, "content": [
  { "type": "decisionItem", "attrs": { "localId": "d1", "state": "DECIDED" },
    "content": [{ "type": "text", "text": "Merged + version bumped + deployed" }] }
] }
```

### Bullet / ordered list

```json
{ "type": "bulletList", "content": [
  { "type": "listItem", "content": [{ "type": "paragraph", "content": [{ "type": "text", "text": "Item" }] }] }
] }
```

Use `"type": "orderedList"` for numbered lists.

### Table

`tableHeader` for header cells, `tableCell` for body. Cell content must be block nodes (paragraphs).

```json
{ "type": "table", "attrs": { "isNumberColumnEnabled": false, "layout": "default" }, "content": [
  { "type": "tableRow", "content": [
    { "type": "tableHeader", "content": [{ "type": "paragraph", "content": [{ "type": "text", "text": "Scenario" }] }] },
    { "type": "tableHeader", "content": [{ "type": "paragraph", "content": [{ "type": "text", "text": "Expected" }] }] }
  ] },
  { "type": "tableRow", "content": [
    { "type": "tableCell", "content": [{ "type": "paragraph", "content": [{ "type": "text", "text": "Empty id list" }] }] },
    { "type": "tableCell", "content": [{ "type": "paragraph", "content": [{ "type": "text", "text": "400 validation error" }] }] }
  ] }
] }
```

### Rule (horizontal divider)

```json
{ "type": "rule" }
```

## Inline nodes (live inside a paragraph's `content`)

### Status lozenge (badge pill)

`color`: `neutral` (grey) · `purple` · `blue` · `red` · `yellow` · `green`.

```json
{ "type": "status", "attrs": { "text": "IN PROGRESS", "color": "yellow", "localId": "s1" } }
```

### Date pill (smart relative date)

`timestamp` is **epoch milliseconds as a string**. Convert an ISO date before building the doc
(e.g. `2026-06-11` → `"1781481600000"`).

```json
{ "type": "date", "attrs": { "timestamp": "1781481600000" } }
```

### Emoji

```json
{ "type": "emoji", "attrs": { "shortName": ":rocket:", "text": "🚀" } }
```

Inline unicode emoji in plain `text` (e.g. `"🎯 Objective"`) also works and is simpler.

### Mention

```json
{ "type": "mention", "attrs": { "id": "<account_id>", "text": "@Vitalii" } }
```

### Hard break (line break inside a paragraph)

```json
{ "type": "hardBreak" }
```

### Inline link / marks

Marks decorate `text` nodes:

```json
{ "type": "text", "text": "Open the PR", "marks": [{ "type": "link", "attrs": { "href": "https://..." } }] }
{ "type": "text", "text": "bold", "marks": [{ "type": "strong" }] }
{ "type": "text", "text": "italic", "marks": [{ "type": "em" }] }
{ "type": "text", "text": "code", "marks": [{ "type": "code" }] }
```

## Gotchas

- **Unique `localId`s** — `status`, `taskItem`, `decisionItem`, `taskList`, `decisionList` each need
  one; duplicates render incorrectly. Use stable ids like `ac1`, `ac2`, `d1`.
- **`date.timestamp` is a string of epoch ms**, not an ISO string.
- **Table cells hold block nodes** — wrap text in a `paragraph`, never put `text` directly in a cell.
- **Inline nodes only inside paragraphs** — `status`/`date`/`emoji`/`mention` can't be top-level.
- **`status` text** is uppercased visually by Jira; write it readable, it'll render as a badge.
