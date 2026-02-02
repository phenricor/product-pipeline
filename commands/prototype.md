---
description: Generate UI prototypes as standalone HTML files
workflow: "2/3: extract/analyse → prototype → post"
args:
  id:
    description: The activity ID to generate prototypes for
    required: true
---

## PHASE 0: LOAD CONTEXT

**Detect design system** (check in order):
1. `design-system.json` in repo root
2. `design-system.md` in repo root
3. If none found: use Bootstrap 5 defaults

**Check for learned patterns** in `.claude/skills/patterns/SKILL.md` (if exists).

**Check for learned antipatterns** in `.claude/skills/antipatterns/SKILL.md` (if exists).

**Find UI reference** in the project:
- Search for existing views/templates/components related to the feature
- Extract styles, patterns, and conventions
- If no reference found: use design system + generic patterns

**Read instructions** from `.claude/activities/{{id}}/prototype-prompt.md`

---

## WORKFLOW

### 1. Pre-Analysis (BEFORE generating)

**Search for project references:**
- Locate screens/components similar to what will be prototyped
- Extract CSS/styles used
- Identify layout patterns (Grid, Flex, etc.)

**From design system, extract:**
- Color palette
- Typography (fonts, weights, sizes)
- Standard spacing
- Existing components (buttons, forms, cards, modals)
- Specific notes for prototypes

### 2. Style Hierarchy

```
Project CSS > design-system > Bootstrap/default framework
```

### 3. Fidelity Rules

**DO NOT invent/improve. Replicate what exists:**
- Border-radius: `0` = `0`, not `5px`
- Border-width: `6px` ≠ `4px`
- Colors: keep exact (including opacity)
- Font-weight: `700` ≠ `500`
- Dimensions: `15px` ≠ `20px`
- Layout: Grid ≠ Flex

### 4. Mock Data

**Use realistic data:**
- Lists/loops: 3-5 items
- Conditionals: show relevant state
- Names, values, dates: realistic for context
- Links/actions: use `href="#"` or `onclick="return false"`

### 5. HTML File Structure

**Output always as `.html` with inline CSS and JS:**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{Feature} - Prototype</title>
    <!-- CSS Framework (from design-system or Bootstrap) -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css" rel="stylesheet">
    <style>
        /* Project/design-system CSS */
    </style>
</head>
<body>
    <!--
    Prototype: {feature}
    Reference: {reference_file or "generic design-system"}
    Change: {description}
    -->

    <!-- Content -->

    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
    <script>
        // JavaScript for interactivity
    </script>
</body>
</html>
```

**Modals:** render as open
```html
<div class="modal-backdrop fade show" style="opacity:0.5"></div>
<div class="modal fade show" style="display:block">...</div>
```

### 6. Consolidation

**Prefer ONE interactive HTML file:**
- States of the same UI (view, edit, create, modal) → one file with JS to toggle
- CSS defined only once
- Multiple files only for distinct pages

```html
<script>
let state = 'view'; // view | edit | create
function switchState(newState) {
    state = newState;
    // Toggle visibility of elements
}
</script>
```

### 7. Checklist

```
[ ] Read design-system.json/md?
[ ] Checked .claude/skills/patterns/ and .claude/skills/antipatterns/ for learnings?
[ ] Searched for UI reference in project?
[ ] Maintained fidelity to existing styles?
[ ] Mock data is realistic?
[ ] States consolidated in interactive file?
[ ] Traceability header included?
```

### 8. Output

```
Save: .claude/activities/{{id}}/prototypes/{feature}.html
Reply: ✓ Generated: {name} | Reference: {source}
```

## Warnings

**Alert when:**
- Design system not found → "Using default Bootstrap"
- UI reference not found → "Generic prototype"
- Large file → "Read partially"

**NEVER:**
- "Improve" existing styles
- Invent colors/classes
- Generate files other than .html

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

- `/post {{id}}` - Upload PRD and prototypes to Azure DevOps
