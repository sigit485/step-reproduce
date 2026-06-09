---
name: step-reproduce
description: >
  Generate iOS QA step-to-reproduce documentation from a ticket description and implemented code.
  Use this skill whenever the user asks to "buatkan step reproduce", "buat step to reproduce",
  "generate QA steps", "bikin dokumentasi testing", or pastes a ticket and asks for testing steps.
  Also trigger when the user provides a JIRA/Linear ticket title/description and wants QA docs,
  or says "buatkan template testing untuk tiket ini". The skill reads git log and modified Swift
  files to understand what was actually implemented before writing the steps.
---

# Step-to-Reproduce Generator

Generate structured QA step-to-reproduce documentation by analyzing what was actually implemented
in the current branch, then mapping that to the ticket's requirements.

## Process

### 1. Gather context

Run these in parallel:

```bash
# Recent commits on current branch (vs master)
git log master..HEAD --oneline

# Full diff stat to see all changed files
git diff master..HEAD --stat

# Modified/added Swift files content (for the key ones)
git diff master..HEAD --name-only | grep '\.swift$'
```

Read the key changed Swift files — focus on ViewControllers, Views, ViewBuilders, and Models.
Skip test files and project.pbxproj unless needed.

### 2. Understand what was built

From the code, identify:
- **What feature** was added/changed (UI elements, data fields, behavior)
- **Where** it appears (which screen, which section, triggered by what)
- **What modules** are affected (e.g., Approval, Profile, Dashboard, etc.)
- **Any groupings** (e.g., multiple UI sections, multiple detail fields)

Cross-reference with the ticket description the user provided to fill gaps.

### 3. Write the documentation

Output a complete markdown block. Use Indonesian for all labels, steps, and descriptions.
Use the exact structure below — don't skip sections.

---

## Output Template

```markdown
## 📋 Steps to Reproduce — [TICKET-ID]
**[Ticket Title]**

---

### 🔧 Preconditions
- [Role requirement, e.g.: Akun dengan role User Approval]
- [Data requirement, e.g.: Terdapat data Leave Request dengan variasi Branch dan Department]
- [Build requirement, e.g.: Build dari branch `branch-name`]

---

### 🪜 Steps to Reproduce

#### Bagian 1 — [Feature Area / Module Name]

1. [Action step]
2. [Action step]
3. [Observation step — "Observasi apakah..."]
...

#### Bagian 2 — [Second Feature Area if applicable]

1. ...

---

### ✅ Expected Results

| # | Ekspektasi |
|---|-----------|
| 1 | [Expected behavior 1] |
| 2 | [Expected behavior 2] |
...

---

### 🔍 Technical Notes (untuk reviewer)
- `ComponentName` sekarang [description] di [`path/to/file.swift`](path/to/file.swift)
- [Protocol/contract info if relevant]
- [Helper/utility info if relevant]
- [Module differences if relevant, e.g.: "Leave pakai 4 sections; Claim pakai 3 sections"]
```

---

## Writing Guidelines

**Steps**: Write as actions + observations alternating. Actions: "Tap X", "Scroll ke Y", "Login sebagai Z".
Observations: "Observasi apakah X tampil", "Pastikan Y sesuai dengan Z".

**Grouping**: If the feature touches multiple modules (e.g., Leave AND Claim), create separate
Bagian sections. If it's one module with sub-flows (e.g., list → detail), still split by sub-flow
when there are 5+ steps per flow.

**Preconditions**: Always include role, data, and build requirements. Be specific about what data
is needed — "employee yang memiliki Branch dan Department" not just "data ada".

**Expected Results table**: Cover both happy path (things that should appear) AND integrity checks
(data should match source, layout should not break). Typically 5–8 rows.

**Technical Notes**: List key files with markdown links using relative paths from project root.
Name the component/protocol/helper. Note any per-module differences (e.g., Leave has X sections,
Claim has Y sections). This section is for code reviewers, not QA.

**Tone**: Factual and direct. No filler. Steps should be unambiguous — a QA engineer who has
never seen the code should be able to follow them.

## Example

For a feature that adds filter chip sections (Category, Employee, Branch, Department) to two approval modules:

**Bagian 1** covers Module A filter: open module, tap Filter, observe chip sections,
tap chips, observe selected state + sort behavior, apply filter, verify list, reopen filter to check persist, reset.

**Bagian 2** covers Module B filter: same flow but with fewer chip sections than Module A.

**Technical Notes** would reference the reusable component file, note the protocol/contract used,
and call out per-module differences (e.g., Module A has 4 sections; Module B has 3 sections).
