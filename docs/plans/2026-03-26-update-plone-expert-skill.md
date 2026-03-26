# Plone Expert Developer Skill Update Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Update the `plone-expert-developer` skill to leverage the new official Plone LLM documentation (llms.txt) while removing duplicated reference content.

**Architecture:** We will modify the existing `SKILL.md` file in place. We will add a section at the top to instruct the agent to use the `webfetch` tool to read the official Plone LLM docs. We will then trim the lengthy reference sections at the bottom of the file and replace them with direct links to the relevant markdown files in the Plone documentation.

**Tech Stack:** Markdown, Plone Documentation (llms.txt)

### Task 1: Add Official Documentation Section

**Files:**
- Modify: `/home/erral/.config/opencode/skills/plone-expert-developer/SKILL.md:36-36`

**Step 1: Write the failing test**
(N/A - this is a documentation update, not code)

**Step 2: Run test to verify it fails**
(N/A)

**Step 3: Write minimal implementation**

Insert the following section before the `## Decision Tree` section (around line 36):

```markdown
## Official Documentation & Knowledge Retrieval

When you need detailed information, API references, or encounter an unknown concept, you MUST use the `webfetch` tool to read the official Plone LLM documentation.

- **For a comprehensive index of all documentation:** Fetch `https://6.docs.plone.org/llms.txt` to find the exact markdown file URL you need.
- **For the full, concatenated documentation (use with caution, very large):** Fetch `https://6.docs.plone.org/llms-full.txt`.

Always prefer fetching the specific markdown file from the index over guessing or relying on outdated knowledge.
```

**Step 4: Run test to verify it passes**
(N/A)

**Step 5: Commit**
(N/A - skill files are not in a git repo we manage here)

### Task 2: Trim 'Reference: Field Types and Widgets'

**Files:**
- Modify: `/home/erral/.config/opencode/skills/plone-expert-developer/SKILL.md:591-673`

**Step 1: Write the failing test**
(N/A)

**Step 2: Run test to verify it fails**
(N/A)

**Step 3: Write minimal implementation**

Replace the entire `## Reference: Field Types and Widgets` section (lines 591-673) with the following:

```markdown
## Reference: Field Types and Widgets

Do not guess field types or widgets. Fetch the official documentation for the complete, up-to-date lists:

- **Backend Fields (zope.schema):** Fetch `https://6.docs.plone.org/_sources/backend/fields.md`
- **Volto Schema Widgets:** Fetch `https://6.docs.plone.org/_sources/volto/reference/widgets.md`
```

**Step 4: Run test to verify it passes**
(N/A)

**Step 5: Commit**
(N/A)

### Task 3: Trim 'Reference: Behavior Catalog'

**Files:**
- Modify: `/home/erral/.config/opencode/skills/plone-expert-developer/SKILL.md:674-717`

**Step 1: Write the failing test**
(N/A)

**Step 2: Run test to verify it fails**
(N/A)

**Step 3: Write minimal implementation**

Replace the entire `## Reference: Behavior Catalog` section (lines 674-717) with the following:

```markdown
## Reference: Behavior Catalog

Behaviors are reusable components that add fields and functionality to content types. Activate them in the content type's XML definition. When a user asks for a field, check the official behavior catalog first before defining a new schema field.

- **Complete Behavior Catalog:** Fetch `https://6.docs.plone.org/_sources/backend/behaviors.md`

**Important Note on Default Behaviors:**
Always inspect the `.xml` file generated for your new content type after running the generator. The file (typically `profiles/default/types/MyType.xml`) lists auto-included behaviors. This prevents duplicating fields or functionality.
```

**Step 4: Run test to verify it passes**
(N/A)

**Step 5: Commit**
(N/A)

### Task 4: Trim 'Reference: Volto Block Patterns'

**Files:**
- Modify: `/home/erral/.config/opencode/skills/plone-expert-developer/SKILL.md:718-757`

**Step 1: Write the failing test**
(N/A)

**Step 2: Run test to verify it fails**
(N/A)

**Step 3: Write minimal implementation**

Replace the entire `## Reference: Volto Block Patterns` section (lines 718-757) with the following:

```markdown
## Reference: Volto Block Patterns

For detailed information on creating and configuring Volto blocks, refer to the official documentation.

- **Blocks Anatomy & Registration:** Fetch `https://6.docs.plone.org/_sources/volto/blocks/anatomy.md`
- **Blocks Developer Notes:** Fetch `https://6.docs.plone.org/_sources/volto/blocks/core/index.md`

*(Note: See the "Frontend Scenario Catalog" above for quick templates for custom blocks, variations, and schema enhancers.)*
```

**Step 4: Run test to verify it passes**
(N/A)

**Step 5: Commit**
(N/A)
