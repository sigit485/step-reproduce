# step-reproduce

> A Claude Code skill that auto-generates QA step-to-reproduce documentation
> from your ticket description and implemented code.

Supports **iOS**, **Android**, **Web**, and **Backend** projects. Auto-detects platform
from your file extensions.

## What it does

Reads your current branch's git log and changed source files, maps them to the
ticket requirements you paste in, and produces a structured QA document —
ready to copy-paste into Jira or Linear.

**Output sections:**
- 🔧 **Preconditions** — role, data, and build/environment requirements
- 🪜 **Steps to Reproduce** — grouped by feature area, action + observation format
- ✅ **Expected Results** — table covering happy path + integrity checks
- 🔍 **Technical Notes** — key files and component notes for code reviewers

## Platform support

| Platform | Detected by | Files analyzed |
|----------|------------|----------------|
| iOS | `.swift` files | ViewControllers, Views, ViewBuilders, Models |
| Android | `.kt` / `.java` files | Activities, Fragments, ViewModels, Adapters |
| Web | `.ts` / `.tsx` / `.js` / `.jsx` files | Components, Pages, Hooks, Services |
| Backend | `.go` / `.py` / `.java` / `.cs` / `.rb` files | Controllers, Services, Repositories, Models |

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

1. Check out your feature branch (commits must be in)
2. Open Claude Code in your project
3. Run `/step-reproduce` and paste your ticket description
4. Claude detects platform → reads changed files → generates the full doc

**Trigger phrases:**
- `generate step to reproduce`
- `create QA steps for this ticket`
- `buatkan step reproduce` *(Indonesian)*
- `generate QA steps`

## Output example

```
## 📋 Steps to Reproduce — TICKET-12345
**[Feature: Add Filter Chip Section to Approval Module]**

### 🔧 Preconditions
- Account with Approver role
- Data exists with Branch and Department variants
- Build from branch `feature/my-branch`

### 🪜 Steps to Reproduce

#### Part 1 — Module A Filter
1. Open the app and log in as Approver
2. Navigate to Approval → Module A
3. Tap the Filter icon
4. Observe whether chip sections are displayed
5. Tap a chip → observe selected state
6. Observe that selected chips move to the front (sorted)
7. Apply filter → verify list is filtered correctly
8. Re-open filter → verify chip state persists
9. Tap Reset → verify all chips return to unselected

### ✅ Expected Results
| # | Expected |
|---|---------|
| 1 | Chip sections render correctly |
| 2 | Tapping a chip toggles selected state |
| 3 | Selected chips sort to front of section |
| 4 | Filter applies and list updates |
| 5 | Last filter persists on re-open |
| 6 | Reset clears all selections |

### 🔍 Technical Notes (for reviewers)
- `FilterChipSection` — reusable component at `WMS/View/FilterChipSection.swift`
- `FilterChipItem` — protocol contract; all models conform via extension
- Module A has 4 sections; Module B has 3 (no Type chip)
```

> **Note:** Output language is **Indonesian** by default. The example above is translated
> for illustration — actual output will use Indonesian labels and step phrasing.

## Requirements

- [Claude Code](https://claude.ai/code) installed
- Git repository with feature branch history
- Supported platform (iOS / Android / Web / Backend)

## Output language

Steps and labels are generated in **Indonesian** by default — designed for Indonesian QA teams.
Technical Notes use Indonesian prose with English code identifiers (file paths, class names).

## License

MIT
