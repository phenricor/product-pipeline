# product-pipeline

A Claude Code plugin for product management workflow automation with Azure DevOps integration.

## Overview

This plugin provides commands to streamline the product management workflow:

1. **Extract** user stories from Azure DevOps or **Analyse** user input
2. **Generate** PRDs and UI prototypes
3. **Post** documentation to Azure DevOps
4. **Learn** from feedback to improve future outputs

## Quick Start

## Installation

**Recommended: Install locally** in each project where you want to use it. This keeps learnings (`.claude/skills/`) and activities (`.claude/activities/`) scoped to each project.

### Option 1: Manual Installation

```bash
# Clone the repository
git clone https://github.com/phenricor/product-pipeline /tmp/product-pipeline

# Copy commands and skills to your project
cp -r /tmp/product-pipeline/.claude/commands/* .claude/commands/product/
cp -r /tmp/product-pipeline/.claude/skills/* .claude/skills/product/

# Clean up
rm -rf /tmp/product-pipeline
```

To update the plugin, repeat the steps above.

### Option 2: Plugin Marketplace

```bash
# Add to your local marketplace
plugin marketplace add https://github.com/phenricor/product-pipeline

# Install in your project
plugin install phenricor/product-pipeline
```

## Workflow

```
/extract  ─┐
                            ├→ /prototype → /post
/analyse ─┘
```

| Step | Command | Description |
|------|---------|-------------|
| 1/3 | `/extract` | Extract Work Item from Azure DevOps, generate PRD |
| 1/3 | `/analyse` | Analyse user input, generate PRD (no Azure DevOps) |
| 2/3 | `/prototype` | Generate UI prototypes as HTML files |
| 3/3 | `/post` | Upload PRD and prototypes to Azure DevOps |

## Continuous Learning

The plugin learns from your feedback during each workflow. When you provide corrections:

- **Patterns** (what works) → Appended to `.claude/skills/patterns/SKILL.md`
- **Antipatterns** (what to avoid) → Appended to `.claude/skills/antipatterns/SKILL.md`

These learnings are stored as Claude Code skills in your project and automatically consulted in future runs.

## Commands

### `/extract <workItemId>`

Extracts an Azure DevOps Work Item and generates:
- `.claude/activities/{workItemId}/brief.md` - Raw extracted data
- `.claude/activities/{workItemId}/context.md` - PRD Part 1 (Context & User Flow)
- `.claude/activities/{workItemId}/requisites.md` - PRD Part 2 (Business Rules, Interface Feedback, Edge Cases)
- `.claude/activities/{workItemId}/prototype-prompt.md` - Instructions for prototype generation

### `/analyse <id> [input]`

Analyses user input (direct text or file path) and generates the same PRD structure. Works without Azure DevOps.

### `/prototype <id>`

Generates HTML prototypes following the project's design system. Outputs to `.claude/activities/{id}/prototypes/`.

### `/post <id> [--type] [--parentId]`

Posts the PRD to Azure DevOps:
- `context.md` → Description field
- `requisites.md` → Acceptance Criteria field
- `prototypes/*.html` → Attachments

Creates a new work item if it doesn't exist.

## Setup

### 1. Configure Azure DevOps MCP Server (Optional)

Only needed for `/extract` and `/post` commands.

Add to `.claude/settings.local.json`:

```json
{
  "mcpServers": {
    "azure-devops": {
      "command": "DevOpsMcpPAT",
      "args": ["--org", "YOUR_ORG", "--project", "YOUR_PROJECT", "--pat", "YOUR_PAT"]
    }
  }
}
```

### 2. Add Design System (Optional)

Create `design-system.json` or `design-system.md` in your repository root for consistent prototypes.

### 3. Update .gitignore

```
.claude/activities/
.claude/settings.local.json
```

Note: `.claude/skills/` should be committed so learnings are shared across the team.

## Directory Structure

After running commands, your project will have:

```
your-project/
├── .claude/
│   ├── settings.local.json      # MCP config (do not commit)
│   ├── activities/
│   │   └── {id}/
│   │       ├── brief.md
│   │       ├── context.md
│   │       ├── requisites.md
│   │       ├── prototype-prompt.md
│   │       └── prototypes/
│   │           └── *.html
│   └── skills/
│       ├── patterns/
│       │   └── SKILL.md         # Learned good patterns (skill file)
│       └── antipatterns/
│           └── SKILL.md         # Learned mistakes to avoid (skill file)
├── design-system.json           # Design system (optional)
└── ...
```

## Usage Examples

### From Azure DevOps

```bash
/extract 12345
/prototype 12345
/post 12345
```

### From User Input

```bash
/analyse logout-button "Add logout button to header"
/prototype logout-button
/post logout-button --type "User Story"
```

## License

MIT
