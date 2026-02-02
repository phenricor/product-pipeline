---
description: Post PRD and prototypes to Azure DevOps Work Item (creates if not exists)
workflow: "3/3: extract/analyse → prototype → post"
args:
  id:
    description: The activity ID to post (or Work Item ID if updating existing)
    required: true
  type:
    description: "Work item type for new items (default: User Story)"
    required: false
  parentId:
    description: "Parent work item ID for new items (optional)"
    required: false
---

Post the PRD and prototypes to Azure DevOps Work Item.

If the work item doesn't exist, a new one will be created.

This command completes the cycle started by `/extract` or `/analyse` and `/prototype`.

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
- Activities: `.claude/activities/{{id}}/`
- Prototypes: `.claude/activities/{{id}}/prototypes/`
- Learnings (skills): `.claude/skills/patterns/SKILL.md` and `.claude/skills/antipatterns/SKILL.md`

---

## PHASE 1: VALIDATE FILES EXIST

### Step 1.1: Check Required Files

Verify these files exist before proceeding:
- `.claude/activities/{{id}}/context.md` (required)
- `.claude/activities/{{id}}/requisites.md` (required)

If any required file is missing, abort and inform the user which file is missing.

### Step 1.2: Check Prototype Files

List all HTML files in `.claude/activities/{{id}}/prototypes/` folder that should be attached.
Use glob pattern: `.claude/activities/{{id}}/prototypes/*.html`

### Step 1.3: Read Brief for Title

Read `.claude/activities/{{id}}/brief.md` to extract the title for potential work item creation.

---

## PHASE 2: CHECK WORK ITEM EXISTS

### Step 2.1: Try to Fetch Work Item

```
Tool: mcp__{detected_server}__wit_get_work_item
Parameters:
  id: {{id}}
  expand: "all"
```

### Step 2.2: Determine Action

**If work item exists:** Proceed to PHASE 3 (Update)

**If work item does NOT exist:** Proceed to PHASE 2A (Create)

---

## PHASE 2A: CREATE NEW WORK ITEM

### Step 2A.1: Read PRD Files

Read the content of:
1. `.claude/activities/{{id}}/context.md` - Will go to **Description** field
2. `.claude/activities/{{id}}/requisites.md` - Will go to **Acceptance Criteria** field

### Step 2A.2: Format Content

**Important:** Azure DevOps fields require HTML, not Markdown.
- Convert Markdown to HTML before sending
- Use `<b>` for bold, `<ul><li>` for lists, `<br>` for line breaks

### Step 2A.3: Create Work Item

```
Tool: mcp__{detected_server}__wit_create_work_item
Parameters:
  type: "{{type}}" (default: "User Story")
  title: "<title from brief.md>"
  description: "<html content from context.md>"
  acceptanceCriteria: "<html content from requisites.md>"
  parentId: "{{parentId}}" (if provided)
```

### Step 2A.4: Store New Work Item ID

Save the newly created work item ID for:
- Attaching prototypes
- Updating the local folder name (optional)
- Reporting to user

Proceed to PHASE 4 (Attach Prototypes) with the new ID.

---

## PHASE 3: UPDATE EXISTING WORK ITEM

### Step 3.1: Read PRD Files

Read the content of:
1. `.claude/activities/{{id}}/context.md` - Will go to **Description** field
2. `.claude/activities/{{id}}/requisites.md` - Will go to **Acceptance Criteria** field

### Step 3.2: Format Content

**Important:** Azure DevOps Description field requires HTML, not Markdown.
- If source files are Markdown, convert to HTML before sending
- Use `<b>` for bold, `<ul><li>` for lists, `<br>` for line breaks

### Step 3.3: Update Work Item Using MCP Server

```
Tool: mcp__{detected_server}__wit_update_work_item
Parameters:
  id: {{id}}
  fields: {
    "System.Description": "<html content from context.md>",
    "Microsoft.VSTS.Common.AcceptanceCriteria": "<html content from requisites.md>"
  }
```

---

## PHASE 4: ATTACH PROTOTYPE FILES

### Step 4.1: Upload Each Prototype

For each HTML file found in prototypes folder, use the MCP server to attach:

```
Tool: mcp__{detected_server}__wit_attach_file
Parameters:
  workItemId: <work_item_id> (existing or newly created)
  filePath: "<absolute path to prototype file>"
  comment: "Prototype generated via Claude Code"
```

### Step 4.2: Handle Errors

If an attachment fails:
- Log the error
- Continue with remaining files
- Report failed attachments at the end

---

## PHASE 5: CONFIRMATION

### Step 5.1: Verify Update

Use the MCP server to confirm fields were updated:

```
Tool: mcp__{detected_server}__wit_get_work_item
Parameters:
  id: <work_item_id>
  expand: "all"
```

### Step 5.2: Report Summary

**If work item was CREATED:**
```
## Post Complete (New Work Item Created)

**Work Item:** #<new_id>
**Type:** {{type}}
**Link:** [provided by MCP response]

### Fields Set:
- Title ← brief.md
- Description ← context.md
- Acceptance Criteria ← requisites.md

### Attachments:
- [list of attached prototype files]

### Status: Success/Partial (if some attachments failed)
```

**If work item was UPDATED:**
```
## Post Complete (Work Item Updated)

**Work Item:** #{{id}}
**Link:** [provided by MCP response if available]

### Fields Updated:
- Description ← context.md
- Acceptance Criteria ← requisites.md

### Attachments:
- [list of attached prototype files]

### Status: Success/Partial (if some attachments failed)
```

---

## ERROR HANDLING

- If MCP server not found → Abort with configuration suggestion
- If required files missing → Abort and list missing files
- If API call fails → Retry once, then report error
- If attachment fails → Continue with others, report at end

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

   **Learned from:** Activity {{id}}
   ```

   **For antipatterns** → Append to `.claude/skills/antipatterns/SKILL.md`:
   ```markdown

   ## [Short Title]

   **Context:** [When this applies]

   **Antipattern:** [What to avoid]

   **Instead:** [What to do instead]

   **Learned from:** Activity {{id}}
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

Start by detecting the Azure DevOps MCP server from available tools, then validate that the required files exist.
