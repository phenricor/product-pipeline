---
description: Analyse user input and generate a PRD (no Azure DevOps required)
workflow: "1/3: analyse → prototype → post"
args:
  id:
    description: Unique identifier for this activity (used as folder name)
    required: true
  input:
    description: Feature description (direct text or file path starting with ./ or /)
    required: false
---

Analyse the provided input and generate a complete PRD.

This command works without Azure DevOps - useful for new features, brainstorming, or when working offline.

**Input:** {{input}}

---

## PHASE 0: LOAD INPUT

### Step 0.1: Determine Input Source

**If `input` argument is provided:**
- If starts with `./` or `/` or ends with `.md`/`.txt` → Read file content
- Otherwise → Use as direct text input

**If `input` argument is NOT provided:**
- Ask the user: "Please describe the feature you want to specify, or provide a file path."

### Step 0.2: Validate Input

Ensure the input contains enough information to generate a PRD:
- Feature description or problem statement
- Expected behavior or desired outcome

**If input is too vague:**
- Ask clarifying questions before proceeding

### Step 0.3: Setup Activity Folder

**Local paths (relative to current working directory):**
- Activities: `.claude/activities/{{id}}/`
- Prototypes: `.claude/activities/{{id}}/prototypes/`
- Learnings (skills): `.claude/skills/patterns/SKILL.md` and `.claude/skills/antipatterns/SKILL.md`

Create the folder structure:
```bash
mkdir -p .claude/activities/{{id}}/prototypes .claude/skills/patterns .claude/skills/antipatterns
```

---

## PHASE 1: CREATE BRIEF

### Step 1.1: Parse Input

Extract from the user's input:
- **Title**: Main feature name (infer if not explicit)
- **Description**: What the feature should do
- **Context**: Why this is needed (if provided)
- **Acceptance Criteria**: Success conditions (if provided)
- **References**: Any mentioned files, screens, or existing features

### Step 1.2: Save Brief

Write parsed data to `.claude/activities/{{id}}/brief.md` with:
- Activity ID
- Title
- Description (original input, cleaned up)
- Extracted acceptance criteria (if any)
- References to existing system (if mentioned)

Use the same language as the user's input for the PRD output.

---

## PHASE 2: GENERATE PRD

### Step 2.1: Load PRD Template and Check Learnings

**Use the `prd-template` skill** to get PRD generation guidelines.

**Check for learned patterns** in `.claude/skills/patterns/SKILL.md` (if exists).

**Check for learned antipatterns** in `.claude/skills/antipatterns/SKILL.md` (if exists).

### Step 2.2: Analyze the Brief

Review the brief, paying special attention to:
- The core problem or need
- Expected user behavior
- Any mentioned constraints or requirements

### Step 2.3: Ask Clarifying Questions (if needed)

Be rigorous: Ask questions and suggest optimizations if needed, especially if there are:
- Ambiguous requirements
- Missing edge cases
- Potential UX issues
- Unclear business rules

**Index all questions with numbers** for easy reference.

### Step 2.4: Generate PRD

Generate the PRD following the structure from prd-template skill.

**Key principles:**
- Be proportional to complexity (simple requests = concise PRD)
- Never assume - ask when unclear
- Include complete message examples where applicable

### Step 2.5: Split and Save Output

Create two files in `.claude/activities/{{id}}/`:

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

Provide commands to generate a prototype. It should be concise but organize the criteria to build it. Consider buttons, components (modal, confirmations, toasts) and other UI parts. From the brief, extract references from the current system state if mentioned. Example: "In the New Item modal in Quotes, include a field called Value. This field should be monetary and required. When it has a result, show a Currency selector field with USD and BRL options." If it has multiple impact areas, separate them by headings, like Page and Modal. The output must be created in `.claude/activities/{{id}}/prototype-prompt.md`.

---

## PHASE 4: CONFIRM COMPLETION

Provide a summary including:
- Activity ID and title
- Files created:
  - `.claude/activities/{{id}}/brief.md`
  - `.claude/activities/{{id}}/context.md`
  - `.claude/activities/{{id}}/requisites.md`
  - `.claude/activities/{{id}}/prototype-prompt.md`

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

## NEXT STEPS

- `/prototype {{id}}` - Generate UI prototypes
- `/post {{id}}` - Create work item in Azure DevOps (if MCP server available)

---

Start by determining the input source and validating the content.
