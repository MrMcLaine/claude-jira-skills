# claude-jira-skills

A growing collection of [Claude](https://claude.com/claude-code) skills for Jira. The flagship
`jira-create` skill turns plain requests into **dashboard-style tickets** using Atlassian Document
Format (ADF) — colored panels, status lozenges, collapsible sections, interactive checklists, and
tables — instead of flat markdown. More skills (search, triage, sprint reporting) are on the way.

All skills run on the official **Atlassian Remote MCP**, so they work anywhere Claude supports MCP.

## Skills

| Skill         | What it does                                                                 |
| ------------- | ---------------------------------------------------------------------------- |
| `jira-create` | Create rich, dashboard-style Jira issues (Task, Bug, Story, Epic, Subtask)   |

## Layout

```
claude-jira-skills/
├── skills/
│   └── jira-create/
│       ├── SKILL.md
│       └── templates/        # ready-to-fill ADF documents
│           ├── task.adf.json
│           ├── bug.adf.json
│           └── story.adf.json
└── shared/
    └── adf-cheatsheet.md      # reusable ADF reference for any skill
```

## Install

Copy a skill into your Claude skills directory (project- or user-level):

```bash
# project-level
cp -r skills/jira-create <your-repo>/.claude/skills/

# or user-level (available everywhere)
cp -r skills/jira-create ~/.claude/skills/
```

The skill loads its `templates/` and `../../shared/adf-cheatsheet.md` on demand.

## Requirements

- Claude with the official **Atlassian Remote MCP** connected (`mcp__atlassian__*` tools).
- A Jira Cloud site you can create issues in.

## Why ADF?

Markdown descriptions give you headings, lists, and code blocks. ADF unlocks the parts of Jira that
make a ticket actually scannable: colored callout **panels**, **status lozenges** (the badge pills),
**collapsible** sections, **interactive checklists**, **decision lists**, smart **date** pills, and
real **tables**. See [`shared/adf-cheatsheet.md`](./shared/adf-cheatsheet.md).
