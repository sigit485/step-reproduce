# step-reproduce

> A Claude Code skill that auto-generates iOS QA step-to-reproduce documentation
> from your ticket description and implemented code.

## What it does

Reads your current branch's git log and modified Swift files, maps them to the
ticket requirements you paste in, and produces a structured QA document in Indonesian —
ready to copy-paste into Jira/Linear.

**Output sections:**
- 🔧 **Preconditions** — role, data, and build requirements
- 🪜 **Steps to Reproduce** — grouped by feature area, action + observation format
- ✅ **Expected Results** — table covering happy path + integrity checks
- 🔍 **Technical Notes** — file refs and component notes for code reviewers

## Installation

### Option A — Copy to your local skills directory

```bash
cp -r step-reproduce/ ~/.claude/skills/step-reproduce/
```

### Option B — Clone and symlink

```bash
git clone https://github.com/sigit485/step-reproduce.git ~/projects/step-reproduce
ln -s ~/projects/step-reproduce/step-reproduce ~/.claude/skills/step-reproduce
```

## Usage

1. Make sure you're on your feature branch (git commits must be in)
2. Open Claude Code in your project
3. Type `/step-reproduce` and paste your ticket description
4. Claude reads the git log + Swift files → generates the full doc

**Auto-triggers on phrases like:**
- `buatkan step reproduce`
- `buat dokumentasi testing untuk tiket ini`
- `generate QA steps`

## Output example

```
## 📋 Steps to Reproduce — WMSP-12345
**[Feature Name]**

### 🔧 Preconditions
- Akun dengan role User Approval
- Terdapat data dengan variasi Branch dan Department
- Build dari branch `feature/my-branch`

### 🪜 Steps to Reproduce

#### Bagian 1 — Module A
1. Buka aplikasi, login sebagai User Approval
2. Masuk ke menu Approval → Module A
3. Tap icon Filter
4. Observasi apakah chip section tampil...
...

### ✅ Expected Results
| # | Ekspektasi |
|---|-----------|
| 1 | Chip section tampil dengan benar |
...

### 🔍 Technical Notes (untuk reviewer)
- `ComponentName` di `path/to/file.swift`
```

## Requirements

- [Claude Code](https://claude.ai/code) installed
- iOS/Swift project with git history
- Feature branch with commits

## Language

Steps and labels are generated in **Indonesian** by default (suitable for Indonesian QA teams).
The Technical Notes section is in Indonesian with English code references.

## License

MIT
