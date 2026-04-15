---
name: refine-linear-issue
description: Refines a rough issue description into a well-structured Linear ticket using codebase knowledge. Use when the user wants to create, refine, or write a Linear issue, ticket, or task from a description. Accepts either a plain description (creates new issue) or a Linear issue ID/URL (edits existing issue).
---

# Refine Linear Issue

Turn a rough description into a detailed, well-structured Linear issue grounded in the codebase.

## Workflow

### 1. Understand the description

Read the user's input. Determine:
- The problem being solved or the feature being added
- Whether a Linear issue ID or URL was provided (edit mode) or only a description (create mode)

### 2. Explore the codebase

Search for relevant files, functions, and patterns. Look for:
- Files likely to be modified
- Existing patterns or utilities to reuse
- Current behavior that needs to change

### 3. Clarify (only if needed)

If scope, priority, or team ownership is ambiguous, ask 1-2 targeted questions. Skip if intent is clear.

### 4. Draft the issue

Produce a structured issue with:

**Title** — imperative, concise ("Fix X", "Add Y", "Refactor Z")

**Problem / Context**
- Current behavior or gap
- Why it matters
- Relevant codebase references (files/functions)

**Proposed Solution**
- What to implement or change
- Technical approach (patterns to follow, files to modify)

**Acceptance Criteria**
- [ ] Verifiable, testable outcomes
- [ ] Edge cases covered

**Technical Notes** *(optional)*
- File paths, function names, schema changes

**Out of Scope** *(optional)*
- Explicit boundaries

### 5. Create or edit

- **Edit mode** (Linear issue ID/URL provided): fetch the existing issue with `get_issue`, then update it using `save_issue` with the issue ID. Preserve unchanged fields (assignee, status, labels, etc.). Do not ask for confirmation — apply the refinement directly.
- **Create mode** (description only, no ticket reference): present the draft to the user for review, then create a new Linear issue with `save_issue` on approval. Apply labels and set priority if specified.
