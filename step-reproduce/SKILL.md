---
name: step-reproduce
description: >
  Generate QA step-to-reproduce documentation from a ticket description and implemented code.
  Supports iOS, Android, Web, and Backend projects — auto-detects platform from file extensions.
  Use this skill whenever the user asks to "buatkan step reproduce", "buat step to reproduce",
  "generate QA steps", "create step to reproduce", "bikin dokumentasi testing", "generate step
  to reproduce for this ticket", or pastes a ticket and asks for testing steps.
  Also trigger when the user provides a JIRA/Linear ticket title/description and wants QA docs,
  or says "buatkan template testing untuk tiket ini". The skill reads git log and modified source
  files to understand what was actually implemented before writing the steps.
---

# Step-to-Reproduce Generator

Generate structured QA step-to-reproduce documentation by analyzing what was actually implemented
in the current branch, then mapping that to the ticket's requirements.

## Process

### 1. Detect platform + gather context

Run these in parallel:

```bash
# Recent commits on current branch
git log master..HEAD --oneline 2>/dev/null || git log main..HEAD --oneline

# All changed files (to detect platform)
git diff master..HEAD --name-only 2>/dev/null || git diff main..HEAD --name-only

# Diff stat for scope overview
git diff master..HEAD --stat 2>/dev/null || git diff main..HEAD --stat
```

**Detect platform from changed file extensions:**

| Extensions found | Platform |
|-----------------|---------|
| `.swift` | iOS |
| `.kt` or `.java` + `AndroidManifest` | Android |
| `.ts` / `.tsx` / `.js` / `.jsx` | Web |
| `.go` / `.py` / `.java` / `.cs` / `.rb` + no Android manifest | Backend |

If multiple platforms detected (e.g., monorepo), generate separate sections per platform.

### 2. Read key changed files

**iOS** — focus on: ViewControllers, Views, ViewBuilders, Models. Skip: `*.xcodeproj`, `Pods/`, test files.
```bash
git diff master..HEAD --name-only | grep '\.swift$' | grep -v 'Tests\|Spec\|Mock'
```

**Android** — focus on: Activities, Fragments, ViewModels, Adapters, XML layouts. Skip: test files, `build/`.
```bash
git diff master..HEAD --name-only | grep -E '\.(kt|java|xml)$' | grep -v 'Test\|test\|build/'
```

**Web** — focus on: Components, Pages, Hooks, Services, API files. Skip: `*.test.*`, `*.spec.*`, `node_modules/`.
```bash
git diff master..HEAD --name-only | grep -E '\.(ts|tsx|js|jsx)$' | grep -v '\.test\.\|\.spec\.\|node_modules'
```

**Backend** — focus on: Controllers/Handlers, Services, Repositories, Models/Schemas, Routes. Skip: test files.
```bash
git diff master..HEAD --name-only | grep -vE '(_test|\.test\.|spec|mock)' | grep -E '\.(go|py|java|cs|rb|php)$'
```

Read the 3–6 most impactful files. Prioritize files with the most diff lines.

### 3. Understand what was built

From the code, identify:
- **What feature** was added/changed (UI elements, data fields, API endpoints, behavior)
- **Where** it appears (which screen/endpoint/component, triggered by what)
- **What modules/layers** are affected
- **Any groupings** (multiple UI sections, multiple API endpoints, multiple screens)

Cross-reference with the ticket description the user provided to fill gaps.

### 4. Write the documentation

Output a complete markdown block. **Use Indonesian** for all labels, steps, and descriptions.
Apply the platform-specific guidelines below, then use the output template.

---

## Platform-Specific Guidelines

### iOS

**Steps focus on:** UI navigation, tap interactions, scroll behavior, visual observations.
- Actions: `"Buka aplikasi"`, `"Tap tombol X"`, `"Scroll ke bagian Y"`, `"Login sebagai [role]"`
- Observations: `"Observasi apakah X tampil"`, `"Pastikan Y sesuai dengan Z"`
- Preconditions: role, device/iOS version if relevant, test data requirements, build branch
- Technical Notes: Swift file paths, protocol/delegate names, ViewController hierarchy

### Android

**Steps focus on:** app navigation, tap/swipe interactions, RecyclerView behavior, visual observations.
- Actions: `"Buka aplikasi"`, `"Tap pada item X"`, `"Swipe ke kiri pada Y"`, `"Login sebagai [role]"`
- Observations: `"Observasi apakah X muncul"`, `"Pastikan Y menampilkan data Z"`
- Preconditions: role, device/API level if relevant, test data, build variant (debug/release)
- Technical Notes: Kotlin/Java file paths, ViewModel state, Fragment/Activity names

### Web

**Steps focus on:** browser navigation, clicks, form interactions, network requests, visual state.
- Actions: `"Buka browser, akses URL X"`, `"Klik tombol Y"`, `"Isi form field Z dengan [value]"`
- Observations: `"Observasi apakah komponen X tampil"`, `"Pastikan response menampilkan Y"`
- Preconditions: role/permission, browser, test data, environment (staging/production), seed data
- Technical Notes: Component file paths, API endpoint, state management notes (Redux/Zustand/etc.)
- Include network tab verification step if the feature involves API calls

### Backend

**Steps focus on:** API calls (curl/Postman), request/response validation, DB state before/after.
- Actions: `"Kirim request POST ke endpoint X dengan body: {...}"`, `"Jalankan query untuk verifikasi"`
- Observations: `"Observasi response status dan body"`, `"Verifikasi data tersimpan di database"`
- Preconditions: auth token/credentials, test data setup, environment, DB seed state
- Technical Notes: Endpoint paths, service/handler file, DB schema changes, migration notes
- Always include: expected HTTP status code, response body structure, DB side effects

---

## Output Template

```markdown
## 📋 Steps to Reproduce — [TICKET-ID]
**[Ticket Title]**

---

### 🔧 Preconditions
- [Role/permission requirement]
- [Test data requirement — be specific]
- [Build/environment requirement, e.g.: Build dari branch `branch-name`]
- [Platform-specific: device, browser, API key, etc. — only if relevant]

---

### 🪜 Steps to Reproduce

#### Bagian 1 — [Feature Area / Module / Endpoint]

1. [Action step]
2. [Action step]
3. [Observation step — "Observasi apakah..." / "Verifikasi bahwa..."]
...

#### Bagian 2 — [Second Area if applicable]

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
- `ComponentName` — [description] di [`path/to/file`](path/to/file)
- [Protocol/interface/contract info if relevant]
- [Per-module or per-platform differences if relevant]
```

---

## Writing Guidelines

**Steps**: Alternate actions and observations. Actions are imperative ("Tap", "Klik", "Kirim request").
Observations start with "Observasi apakah..." or "Pastikan..." or "Verifikasi bahwa...".

**Grouping**: Split into Bagian sections when the feature touches multiple modules/screens/endpoints,
or when a single flow has 6+ steps (list → detail counts as two flows).

**Preconditions**: Always include role, data, and build/env requirements. Be specific —
`"employee yang memiliki Branch dan Department"` not `"data ada"`.
For Backend: include auth token setup and DB seed state.
For Web: include browser and environment.

**Expected Results table**: 5–8 rows. Cover both happy path AND integrity checks
(data matches source, layout intact, no side effects on unrelated data).

**Technical Notes**: Markdown links to key files. Name components/protocols/endpoints.
Note per-module differences. This section is for code reviewers, not QA testers.

**Tone**: Factual, direct, no filler. A QA engineer who has never seen the code should
be able to follow the steps without ambiguity.
