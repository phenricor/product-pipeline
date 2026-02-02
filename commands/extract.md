---
description: Extract Azure DevOps Work Item to a User Story
workflow: "1/3: extract → prototype → post"
args:
  workItemId:
    description: The Azure DevOps Work Item ID to extract
    required: true
---

I need you to extract the details of Azure DevOps User Story #{{workItemId}}, save them to a brief, and then generate a complete PRD.

---

## PHASE 0: DETECT MCP SERVER

**Auto-detect Azure DevOps MCP server** from available tools in the current project.

Look for tools matching pattern `mcp__*__wit_get_work_item` to identify the MCP server name.

**If no Azure DevOps MCP server found:**

1. Ask the user: "No Azure DevOps MCP server detected. Do you have an existing configuration file with MCP settings? (provide path or 'no')"

2. **If user provides a file path:**
   - Read the file and extract MCP server configuration
   - Guide user to copy relevant settings to `.claude/settings.local.json`

3. **If user says 'no' or no valid config found:**
   - Guide user through setup:
   ```
   To connect to Azure DevOps, create `.claude/settings.local.json`:

   {
     "mcpServers": {
       "azure-devops": {
         "command": "DevOpsMcpPAT",
         "args": ["--org", "YOUR_ORG", "--project", "YOUR_PROJECT", "--pat", "YOUR_PAT"]
       }
     }
   }

   Replace:
   - YOUR_ORG: Your Azure DevOps organization name
   - YOUR_PROJECT: Your project name
   - YOUR_PAT: Personal Access Token (create at https://dev.azure.com/{org}/_usersSettings/tokens)

   After creating the file, restart Claude Code and run this command again.
   ```
   - Abort the command.

**Local paths (relative to current working directory):**
- Activities: `.claude/activities/{{workItemId}}/`
- Prototypes: `.claude/activities/{{workItemId}}/prototypes/`
- Learnings (skills): `.claude/skills/patterns/SKILL.md` and `.claude/skills/antipatterns/SKILL.md`

---

## PHASE 1: EXTRACT FROM AZURE DEVOPS

### Step 1.1: Fetch Work Item Using MCP Server

Use the detected MCP server to fetch the work item:

```
Tool: mcp__{detected_server}__wit_get_work_item
Parameters:
  id: {{workItemId}}
  expand: "all"
```

Extract key fields from the response:
- Title (`System.Title`)
- Description (`System.Description`) - often contains proposed solution
- Acceptance Criteria (`Microsoft.VSTS.Common.AcceptanceCriteria`)
- Customer (`Custom.Customer`)
- Priority (`Microsoft.VSTS.Common.Priority`)
- Relations (Parent, Child, Related)

Use the same language as the work item for the PRD output.

### Step 1.2: Create Brief

Create folder and save brief:
```bash
mkdir -p activities/{{workItemId}}
```

Write extracted data to `.claude/activities/{{workItemId}}/brief.md` with:
- Work Item ID and Type
- Title
- Description (converted from HTML to Markdown)
- Acceptance Criteria (if exists)
- Relations

---

## PHASE 2: GENERATE PRD

### Step 2.1: Load PRD Template and Check Learnings

**Use the `prd-template` skill** to get PRD generation guidelines.

**Check for learned patterns** in `.claude/skills/patterns/SKILL.md` (if exists).

**Check for learned antipatterns** in `.claude/skills/antipatterns/SKILL.md` (if exists).

### Step 2.2: Analyze the Brief

Review the extracted brief, paying special attention to:
- The **Description** field which may contain a proposed solution
- The **Acceptance Criteria** for validation scenarios
- The **Objective** for business context
- Do not explore the codebase yet, leave it for prototyping.
- Avoid wireframes and drawings, prioritize pure and concise writing.

### Step 2.3: Ask Clarifying Questions (if needed)

Be rigorous: Ask questions and suggest optimizations if needed, especially if there are ambiguous parts or bad UX patterns.

### Step 2.4: Generate PRD

Generate the PRD following the structure from prd-template skill.

**Key principles:**
- Be proportional to complexity (simple requests = concise PRD)
- Never assume - ask when unclear
- Include complete message examples where applicable

### Step 2.5: Split and Save Output

Create two files in `.claude/activities/{{workItemId}}/`:

**context.md** - Contains:
- Title (# PRD - [Feature Name])
- ## 1. Context & Value (full section)
- ## 2. User Flow (full section)

**requisites.md** - Contains:
- Title (# PRD - [Feature Name])
- ## 3. Business Rules (full section)
- ## 4. Interface Feedback (full section)
- ## 5. Edge Cases (full section)
- ## 6. Technical Criteria (placeholder)

---

## PHASE 3: PREPARE PROTOTYPE COMMANDS

Provide commands to generate a prototype. It should be concise but organize the criteria to build it. Consider buttons, components (modal, confirmations, toasts) and other UI parts. From the brief, extract references from the current system state. Example: "In the New Item modal in Quotes, include a field called Value. This field should be monetary and required. When it has a result, show a Currency selector field with USD and BRL options." If it has multiple impact areas, separate them by headings, like Page and Modal. The output must be created in `.claude/activities/{{workItemId}}/prototype-prompt.md`.

---

## PHASE 4: CONFIRM COMPLETION

Provide a summary including:
- Work item title and ID
- Azure DevOps link (if available from MCP response)
- Files created:
  - `.claude/activities/{{workItemId}}/brief.md`
  - `.claude/activities/{{workItemId}}/context.md`
  - `.claude/activities/{{workItemId}}/requisites.md`
  - `.claude/activities/{{workItemId}}/prototype-prompt.md`

---

## LEARNINGS: CAPTURE FEEDBACK

**When the user provides corrections or feedback during this workflow:**

1. **Identify the type of learning:**
   - Is this a **pattern** (something that worked well, should be repeated)?
   - Is this an **antipattern** (a mistake to avoid in the future)?

2. **Append to the appropriate skill file** (create directory and file if doesn't exist):

   **For patterns** → Append to `.claude/skills/patterns/SKILL.md`:
   ```markdown

   ## [Short Title]

   **Context:** [When this applies]

   **Pattern:** [What to do]

   **Learned from:** Activity {{workItemId}}
   ```

   **For antipatterns** → Append to `.claude/skills/antipatterns/SKILL.md`:
   ```markdown

   ## [Short Title]

   **Context:** [When this applies]

   **Antipattern:** [What to avoid]

   **Instead:** [What to do instead]

   **Learned from:** Activity {{workItemId}}
   ```

   **If the skill file doesn't exist**, create it with header:
   ```markdown
   # Patterns

   Learned patterns from product-pipeline workflows.
   ```
   or
   ```markdown
   # Antipatterns

   Learned antipatterns to avoid from product-pipeline workflows.
   ```

3. **Confirm with user:** "I've recorded this as a [pattern/antipattern] in `.claude/skills/` for future reference."

---

## NEXT STEPS

- `/prototype {{workItemId}}` - Generate UI prototypes

---

Start by detecting the Azure DevOps MCP server from available tools.
