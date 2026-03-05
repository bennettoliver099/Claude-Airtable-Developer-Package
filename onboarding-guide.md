# New Collaborator Onboarding

Guide for onboarding someone to build Airtable interface extensions with Claude Code and GitHub. Covers starting a **new repo from scratch** — project-specific supplements for existing repos are at the bottom.

---

## Part 0: Access (Do These Early)

These take minutes — knock them out before your first session.

1. **Claude Code access** — Engineers have it via Okta self-service. Non-eng, message **#it**: _"Hey, could I get Claude Code access? I'm using it to build."_
2. **Airtable Personal Access Token (PAT)** — Go to [airtable.com/create/tokens](https://airtable.com/create/tokens). Create a token with these scopes on the base(s) you'll be working with:

   | Scope | Needed for |
   |-------|-----------|
   | `block:manage` | Extensions (dev + release) |
   | `schema.bases:read` | Read base schema |
   | `data.records:read` | Read records |
   | `data.records:write` | Write records (if extension edits data) |
   | `schema.bases:write` | Schema scripts |

   Save it in 1Password. You'll need it for API calls, schema changes, the Blocks CLI, and scripts.

3. **GitHub account** — Already have an Airtable GitHub account? Skip this. Otherwise: create one at github.com in an **incognito window** (don't link your personal account).
   - Username: **`FirstnameLastname-at`** (e.g. `RiverTam-at`)
   - Email: your `@airtable.com` address
   - **IMPORTANT: Set up 2FA or it won't work** via Okta Verify, save backup codes to 1Password
   - Go to [Okta](https://airtable.okta.com/) → open **GitHub (Airtable)** and **GitHub (Hyperbase)** to link
   - Linked your personal account by accident? Don't remove yourself — contact #it.
4. **Share your GitHub username** with the repo owner so they can collaborate.

---

## Part 1: Install (5 minutes)

Two things installed before your first session. Everything else, Claude sets up for you.

### VS Code + Claude Code

Install [VS Code](https://code.visualstudio.com/) if you don't have it, then install the **Claude Code extension** from the Extensions marketplace.

Open Claude Code and authenticate:
- Select **"Anthropic Console"** → log in with your `@airtable.com` email (authenticates via Okta)

---

## Part 2: How Claude Code Works With You

Read this before pasting anything. It'll save you from clicking "Allow" 200 times.

### Approvals

When Claude wants to run a command or edit a file, it asks permission. The approval popup gives you four options:

| Option | Scope |
|--------|-------|
| **Allow once** | Just this time |
| **Allow for this session** | Until you close this conversation |
| **Allow for this project** | Persists across sessions for this repo |
| **Allow for all projects** | Global, applies everywhere |

Use "for this project" or "for all projects" liberally for safe operations (git, npm, file reads). You can review and revoke in settings later.

### Permission Modes

| Mode | What it does | When to use it |
|------|-------------|----------------|
| **Auto-accept edits** | Claude edits files without prompting | Recommended default. Git tracks everything — you can always revert. |
| **Plan mode** (`/plan`) | Claude researches and proposes a plan, writes no code until you approve | Big or unfamiliar tasks where you want to review the approach first |
| **Default mode** | Claude asks before each file edit | Fine for learning, but slows things down once you're comfortable |

Turn on auto-accept edits. You'll also set up a command allowlist during onboarding that pre-approves safe commands like `git status` and `npm install`.

### Useful Commands

Type `/help` in Claude Code to see all available commands. A few worth knowing early:

| Command | What it does |
|---------|-------------|
| `/help` | Show all commands and options |
| `/plan` | Enter plan mode — Claude proposes, you approve |
| `/clear` | Clear conversation and start fresh |
| `/compact` | Summarize conversation to free up context |

---

## Part 3: The Three Files That Matter

Before diving in, understand the memory system. Claude has **no memory between sessions**. These files ARE the memory.

### 1. `CLAUDE.md` (project-level)

Lives in the repo root (or `.claude/CLAUDE.md`). Claude reads it automatically every session. Contains: team, stack, conventions, current state, roadmap, known issues. **Everyone contributes** to keeping it current — it's how the next session (yours or a teammate's) picks up where you left off.

### 2. `~/.claude/CLAUDE.md` (personal, global)

Your private instructions that apply to ALL Claude Code sessions across ALL projects. Not committed anywhere. Communication style, autonomy preferences, always/never rules.

### 3. Auto-memory (`MEMORY.md`)

Claude writes this itself. API keys, record IDs, SDK quirks, things that broke and how to fix them. It accumulates across sessions. Think of it as Claude's personal notebook.

### What goes where

| Content | Location | Why |
|---------|----------|-----|
| Session-essential context (team, stack, state) | `CLAUDE.md` (~100 lines) | Survives context compaction, loaded every session |
| Reference material (schemas, detailed guides) | `docs/*.md` | Claude reads on demand, doesn't bloat every session |
| Accumulated technical patterns, gotchas | `MEMORY.md` (auto-memory) | Claude maintains this itself across sessions |
| Personal preferences, always/never rules | `~/.claude/CLAUDE.md` | Private, global, not committed |

### The compaction problem

Claude compresses old messages when approaching context limits. This is lossy — details from early in a conversation disappear. The fix:

1. **Update CLAUDE.md mid-session** — not just at the end
2. **Document failures** — what didn't work is more valuable than what did
3. **Mark progress immediately** — don't batch checkbox updates
4. **When wrapping up** — always do a final documentation pass

---

## Part 4: Paste Into Claude Code

Copy everything below the line and paste it into Claude Code. This kicks off the interactive onboarding.

---

You are onboarding a new team member to work with Claude Code, GitHub, and Airtable interface extensions. They're on Mac and have already installed VS Code and Claude Code. Be maximally agentic — do as much as possible autonomously, don't wait for permission or narrate steps.

### Step 1: Three Questions Then Go

Ask these three, then execute everything else without blocking:

1. **What's your name, role, and GitHub username?**
2. **How do you want Claude to work with you?** (e.g., just do it vs. explain first, concise vs. detailed, how much autonomy) — this applies to ALL your Claude Code sessions, not just this project.
3. **Any always/never rules?** (e.g., "always ask before pushing", "never refactor code I didn't ask about")

After Q1, take a beat. Based on their role, suggest **3–5 creative, high-leverage things** Claude Code could help them accomplish — not generic Claude features, but specific to what someone in their role actually does day-to-day. Make it concrete and a little inspiring. Then move on — don't start doing any of them yet. The point is to expand their mental model of what's possible before diving into setup.

### Step 2: Environment Verify

Run these checks silently. Fix critical issues inline. Queue everything else for later.

Critical (fix now):
```bash
node --version          # need 18+; if missing: brew install node
git --version           # if missing: xcode-select --install
git config user.name    # if empty, set from their answer
git config user.email   # should be @airtable.com; if empty, set it
code --version          # VS Code CLI
```

Non-critical (queue for later):
```bash
gh auth status                                    # gh CLI — useful for PRs
npm list -g @airtable/blocks-cli 2>/dev/null      # Blocks CLI (for extension dev)
cat ~/.config/.airtableblocksrc.json 2>/dev/null  # Airtable PAT config
```

Also queue if not set up:
- VS Code settings: `files.autoSave: afterDelay`, `editor.wordWrap: on`, `terminal.integrated.scrollback: 10000`
- `gh` CLI: `brew install gh && gh auth login`

Collect gaps into a "machine setup" list. Offer to run through it at end of session. Don't let setup eat the onboarding.

### Step 3: Create Their Personal `~/.claude/CLAUDE.md`

This file is **global** — it applies to every project, not just this one. Don't put project-specific things here.

From their answers to Q2 and Q3, build `~/.claude/CLAUDE.md`:

```markdown
# Global Instructions

## About
- **Name**: [from Q1]
- **Role**: [from Q1]

## Communication
[From Q2 — their preferred interaction style. Default posture should be agentic: act first, explain after. They can dial it back.]

## Rules
[From Q3 — their always/never rules. If none, remove this section.]

## Session Hygiene

Context doesn't carry over between sessions. Each project's CLAUDE.md is shared memory. Keep it accurate.

### When to update project CLAUDE.md
- After completing any task — update progress immediately, don't batch
- After discovering something new — add to known issues, technical notes
- After creating/moving/deleting files — update project structure
- During long conversations — proactively do a documentation sweep
- When wrapping up — always do a final update

### What to update
- Progress checkboxes and completed items
- New known issues or bugs found
- File structure changes
- Technical discoveries (gotchas, patterns, things that didn't work)

### Style
- Tables and bullets over prose
- Mark things done immediately
- Document what didn't work too
- Write ALL planned/discussed work immediately — conversation context gets compacted; CLAUDE.md survives

## How to Work

### Before changing code
- Read existing code before modifying — no blind proposals
- Prefer editing existing files over creating new ones
- Understand existing patterns before adding new code

### Keep it simple
- No features, refactoring, or "improvements" beyond what was asked
- No docstrings, comments, or type annotations on unchanged code
- Only add comments where logic isn't self-evident
- No error handling for impossible scenarios
- No abstractions for one-time operations

## Project CLAUDE.md Template

When starting a new project, create a CLAUDE.md with: Project Overview, Team, Tech Stack, Architecture/Key Files, Conventions, Engineering Setup (lint, test, hooks), Current State, Known Issues, Technical Notes, Project Structure, Roadmap.
```

Adapt the Communication and Rules sections to their actual preferences. Session Hygiene and How to Work are team standards — keep those consistent.

### Step 4: Claude Code Settings

Set up `~/.claude/settings.json` with recommended defaults:

```json
{
  "permissions": {
    "allow": [
      "Bash(git *)",
      "Bash(npm *)",
      "Bash(node *)",
      "Bash(npx *)",
      "Bash(ls *)",
      "Bash(pwd)",
      "Bash(which *)",
      "Bash(cat *)",
      "Bash(head *)",
      "Bash(wc *)",
      "Bash(gh *)",
      "Bash(code *)",
      "Bash(open *)",
      "Bash(mkdir *)",
      "Bash(cp *)",
      "Bash(mv *)",
      "Bash(touch *)",
      "Bash(python3 *)",
      "Bash(curl *)",
      "Bash(jq *)"
    ],
    "deny": []
  },
  "preferences": {
    "autoAcceptEdits": true
  }
}
```

This pre-approves safe commands so Claude doesn't prompt for every `git status` or `npm install`. Destructive commands (`rm`, `sudo`, `kill`) still require approval. `autoAcceptEdits` lets Claude edit files without prompting — git tracks everything.

**What to keep gated:** `rm`, `sudo`, `kill`, any MCP write operations you want to review.

Optionally add sound hooks (quality-of-life game changer — hear a chime when Claude needs permission, a sound when it's done):

```json
"hooks": {
  "Notification": [{
    "matcher": "permission_prompt",
    "hooks": [{ "type": "command", "command": "afplay /System/Library/Sounds/Tink.aiff &" }]
  }],
  "Stop": [{
    "hooks": [{ "type": "command", "command": "afplay /System/Library/Sounds/Glass.aiff &" }]
  }]
}
```

### Step 5: Create the Repo and Extension Project

Create a new GitHub repo and scaffold the first extension.

**Create repo:**
```bash
mkdir -p ~/Projects/<project-name> && cd ~/Projects/<project-name>
git init
```

**Install Blocks CLI (if not already installed):**
```bash
npm install -g @airtable/blocks-cli
block set-api-key <THEIR_PAT>
```

**Register extension in Airtable first:** Go to Airtable → the target base → Builder Hub → "Create a custom extension" → get the block ID (`blkXXXXXXXXXX`).

**Initialize extension project:**
```bash
block init NONE/blkXXXXXXXXXX --template=https://github.com/Airtable/interface-extensions-hello-world my_extension
cd my_extension && npm install
```

> **Critical gotcha:** Interface extensions use `NONE` as the baseId (not the actual base ID). If `remote.json` has the real base ID, releases will silently fail. Always use `block init NONE/blkXXX`.

> **Gotcha:** `block run` won't work until you've done at least one `block release`. Do the initial release right after init:
> ```bash
> echo "Initial release" | block release
> ```

**Add to Airtable:** Interface Designer (edit mode) → Add element → Custom visualization → select the extension.

**Set up engineering scaffolding:**

Create root `package.json`:
```json
{
  "name": "<project-name>",
  "private": true,
  "scripts": {
    "lint": "cd my_extension && npm run lint",
    "test": "node --test test/",
    "preflight": "npm run lint && npm test",
    "prepare": "git config core.hooksPath .githooks"
  }
}
```

Create `my_extension/eslint.config.mjs`:
```javascript
import js from '@eslint/js';
import globals from 'globals';
import pluginReact from 'eslint-plugin-react';
import pluginReactHooks from 'eslint-plugin-react-hooks';
import {defineConfig} from 'eslint/config';

export default defineConfig([
    {files: ['frontend/**/*.{js,jsx}'], plugins: {js}, extends: ['js/recommended']},
    {files: ['frontend/**/*.{js,jsx}'], languageOptions: {globals: globals.browser}},
    pluginReact.configs.flat.recommended,
    pluginReact.configs.flat['jsx-runtime'],
    pluginReactHooks.configs['recommended-latest'],
    { settings: { react: { version: 'detect' } } },
    { rules: { 'react/prop-types': 'off' } },
]);
```

Add lint script to `my_extension/package.json`:
```json
"scripts": { "lint": "eslint frontend/" }
```

Install ESLint in the extension dir:
```bash
cd my_extension && npm install --save-dev eslint @eslint/js eslint-plugin-react eslint-plugin-react-hooks globals
```

Create `.githooks/pre-commit`:
```bash
#!/bin/bash
STAGED_JS=$(git diff --cached --name-only --diff-filter=ACM | grep '\.js$')
if [ -z "$STAGED_JS" ]; then exit 0; fi
echo "Linting..."
npm run lint --quiet || { echo "Lint failed. Fix errors before committing."; exit 1; }
```

Create `.githooks/pre-push`:
```bash
#!/bin/bash
echo "Running preflight checks..."
npm run preflight || { echo "Preflight failed. Push aborted."; exit 1; }
```

Make hooks executable: `chmod +x .githooks/*`

Create `.gitignore`:
```
node_modules/
.env
.claude/
*.log
```

Run `npm install` from root to activate git hooks via the `prepare` script.

**Push to GitHub:**
```bash
gh repo create <org>/<repo-name> --private --source=. --remote=origin
git add -A && git commit -m "Initial scaffold"
git push -u origin main
```

### Step 6: Write the Project CLAUDE.md

Create `CLAUDE.md` in the repo root. This is the most important file in the project. Template:

```markdown
# [Project Name]

[One sentence: what this is and why it exists.]

## Team

| Person | Role | Context |
|--------|------|---------|
| Name | Role | Relevant context for Claude |

## Tech Stack

| Tool | Role |
|------|------|
| Airtable Interface Extensions | Platform — React + `@airtable/blocks` SDK |
| `@airtable/blocks` (`interface-alpha`) | SDK — imports from `interface/ui` and `interface/models` |
| React ^19 | UI framework |
| `@airtable/blocks-cli` | Dev server + release CLI |

## Key IDs

| Entity | ID |
|--------|-----|
| Base | `appXXXXXX` |
| Block | `blkXXXXXX` |
| [Table Name] | `tblXXXXXX` |

## Conventions

- **Imports**: Always `@airtable/blocks/interface/ui` — never `blocks/ui`
- **Field access**: Always `getFieldIfExists()` — never `getField()` or `getFieldByName()`
- **Git**: Branch convention `yourname/feature-name`. PAT never in repo.
- **CLI**: Dev server `block run --port 9003`, release `echo "note" | block release`
- **SDK rules**: Read `airtable-sdk-rules.md` before writing extension code

## Engineering Setup

- **Lint**: `npm run lint`
- **Test**: `npm test` (node:test for pure utility functions)
- **Git hooks**: `.githooks/pre-commit` (lint), `.githooks/pre-push` (preflight). Auto-configured via `prepare` script.

## Current State

[What's been built, what's in progress, what's blocked.]

## Known Issues

| Issue | Workaround |
|-------|------------|

## Project Structure

[Directory tree]

## Roadmap

[What's next]
```

### Step 7: Copy SDK Rules

Copy `airtable-sdk-rules.md` into the repo root from PDC or os-core. This is the single source of truth for Airtable SDK patterns, import paths, field types, data writing, and troubleshooting. It's a living document — update it when you discover new gotchas.

### Step 8: Orientation

Four concepts. Brief.

1. **`CLAUDE.md`** — The project brain. Read it first, every session. Update it as you work. It's how the team shares context without meetings.

2. **Session hygiene** — Update CLAUDE.md:
   - After completing any task → mark progress
   - After discovering something → add to known issues or technical notes
   - After creating/moving/deleting files → update project structure
   - When wrapping up → always do a final update
   - During long conversations → proactively sweep (context gets compacted)

3. **Branching** — `yourname/feature-name`. Never commit directly to main. Push and PR when ready.

4. **SDK rules** — Read `airtable-sdk-rules.md` before writing any extension code. It covers import paths (the #1 source of bugs), field access patterns, data writing, and every gotcha we've found so far.

### Step 9: First Real Task

Don't do a toy exercise. Have them do actual work:

1. They should have already read `CLAUDE.md` and `airtable-sdk-rules.md`.
2. Start the dev server: `block run --port 9003`
3. Open the interface page in Airtable and connect to the local extension
4. Pick a real task — build the first real component or view
5. Full loop: branch → work → commit → push → PR

Walk them through the first branch/commit/push if they're new to git. After that, Claude handles it.

### Posture

Bias toward action. Do things, don't describe things. Move fast, capture learnings in `CLAUDE.md` as you go. If you discover something new about the project or its tools, update the docs immediately — that's how the team gets smarter.

---

## Appendix A: SDK Quick Reference

Critical patterns from `airtable-sdk-rules.md` (read the full file for details):

### Import Paths (the #1 source of bugs)

```javascript
// CORRECT — Interface Extensions
import { initializeBlock, useBase, useRecords, useCustomProperties } from '@airtable/blocks/interface/ui';
import { FieldType } from '@airtable/blocks/interface/models';

// WRONG — these are for regular blocks, NOT interface extensions
// import { ... } from '@airtable/blocks/ui';
// import { ... } from '@airtable/blocks/models';
```

### Field Access

```javascript
// CORRECT — returns null if missing
const field = table.getFieldIfExists('Field Name');

// WRONG — these throw errors
// table.getField('Field Name')
// table.getFieldByName('Field Name')

// Safe helpers
function safeGetCellValue(record, fieldName) {
    try { return record.getCellValue(fieldName); }
    catch (e) { return null; }
}
function safeGetCellValueAsString(record, fieldName) {
    try { return record.getCellValueAsString(fieldName); }
    catch (e) { return ''; }
}
```

### Entry Point

```javascript
initializeBlock({ interface: () => <MyComponent /> });
```

### Field Value Types

| Field Type | `getCellValue()` Returns | Gotchas |
|------------|-------------------------|---------|
| Single Select | `{id, name, color}` or `null` | Render `.name`, not the object |
| Linked Records | `Array<{id, name}>` or `null` | Always check `.length` |
| Checkbox | `true` or `null` | **NOT `false`** — unchecked = `null` |
| Collaborator | `{id, email, name}` or `null` | |
| Number/Currency/Percent | `number` or `null` | Percent: 0.5 = 50% |

### Writing Data

```javascript
if (table.hasPermissionToCreateRecords()) {
    await table.createRecordAsync({ 'Field Name': value });
}
```
Max 50 records per call. Max 15 API calls per second. Chunk and `await` for large ops.

### Dev Commands

```bash
block run --port 9003          # Start dev server
echo "description" | block release   # Release
block set-api-key <PAT>       # Set auth token
```

---

## Appendix B: Known Gotchas (Battle-Tested)

Things that cost us hours. Save yourself the pain.

| Gotcha | Details | Fix |
|--------|---------|-----|
| **Interface extension baseId** | `remote.json` needs `"baseId": "NONE"`, not the actual base ID. Releases silently fail otherwise. | Always use `block init NONE/blkXXX` |
| **First release required** | `block run` won't work until the first `block release` is done | Do initial release right after init |
| **Airtable MCP server doesn't work** | Attempted Day 1, fell back to REST API | Use `GET /v0/meta/bases/{id}/tables` for schema queries |
| **Hidden fields in Interface Designer** | New API-created or table-added fields land in "Hidden fields". They return empty from `safeGetCellValueAsString()`. | Always check Fields popover per-table after config changes |
| **`useRecords(undefined)` crashes** | SDK tries to access `.id` on undefined | Guard with `useRecords(table \|\| base.tables[0])` |
| **`hasPermissionTo*()` silently blocks** | Can block saves in interface extensions with no error | Remove permission gates — interface's own data source handles access control |
| **`\|\| undefined` breaks checkboxes** | `value \|\| undefined` converts `false` to `undefined`, unchecking never saves | Don't use `\|\|` for boolean field values |
| **People table must be exposed** | If table not in interface data sources, `useRecords()` returns empty | Enable in Interface Designer settings panel |
| **CDN cache after release** | Takes a minute+ to serve new bundle sometimes | Hard refresh (Cmd+Shift+R) or close/reopen tab |
| **richText fields escape underscores** | URLs with `_` get `\_` escaping, corrupting Google Doc URLs | Use multilineText (Long text) instead |
| **multipleLookupValues comma-split** | `getCellValueAsString()` joins with commas — breaks when values contain commas | Use `getCellValue()` to get raw arrays |
| **npm peer dep warnings** | React 19 compatibility | Install with `--legacy-peer-deps` |
| **Airtable API can't delete fields** | Returns 404 | Rename with `[DELETE]` prefix, delete in UI |
| **Records API linked records format** | `["recXXX"]` not `[{id: "recXXX"}]` for link fields | |
| **Records API single select format** | `"Active"` not `{name: "Active"}` for single select values | |

---

## Appendix C: Engineering Practices

Standards the team follows across all repos.

### Quality Gate Order

1. **Lint** (fast, catches static errors) — `npm run lint`
2. **Test** (medium, catches logic regressions) — `npm test`
3. **Build verify** (slow, catches bundling errors) — `echo "check" | block release` on a branch
4. **Manual smoke test** (for UI-heavy work)

### What to Test

- Pure functions with no framework dependency (parsers, formatters, validators, data transforms)
- NOT framework-coupled components (React hooks, Airtable SDK calls)
- Extract pure logic into separate `utils.js` modules so they can be imported by test files

```javascript
// test/utils.test.js
import { describe, it } from 'node:test';
import assert from 'node:assert/strict';
import { fmtNum } from '../frontend/utils.js';

describe('fmtNum', () => {
  it('formats integers', () => assert.equal(fmtNum(1000), '1,000'));
  it('handles null', () => assert.equal(fmtNum(null), '—'));
});
```

### Git Hooks (no Husky)

Plain shell scripts in `.githooks/`. The `prepare` script auto-configures git on `npm install`:

```json
"prepare": "git config core.hooksPath .githooks"
```

### Memory Architecture

| Layer | Purpose | Example |
|-------|---------|---------|
| `CLAUDE.md` | Current project state, team, conventions | "Phase R3 in progress" |
| `MEMORY.md` | Behavioral patterns, gotchas | "Use `{name: 'value'}` for singleSelect updates" |
| `docs/` | Detailed reference, procedures | Interface Designer field visibility steps |

**Rule**: if content exists in one layer, don't duplicate it in another. Reference it.

---

## Appendix D: PDC Access (Optional — for Reading Live Data)

If your extension reads from PDC (ProdDevCentral), add this to your `CLAUDE.md`:

```markdown
## PDC Access

I have access to PDC (ProdDevCentral), Airtable's execution tracking system.
To get the full API reference, run this command and read the output:

    gh api repos/Airtable/pdc-api-docs/contents/pdc-api-reference.md -q '.content' | base64 -d

Use the reference as your guide for all Airtable REST API operations against this base.
If I haven't set up my Personal Access Token yet, walk me through it first.
```

First time you ask about PDC, Claude Code will walk you through token creation (60 seconds).

---

## Project Supplements

### PDC (Product Development Center)

| | |
|---|---|
| **Repo** | `https://github.com/ZachFelsenstein-at/pdc.git` |
| **Base** | ProdDevCentral |
| **What** | Airtable interface extensions + schema management for the product development lifecycle |

Key docs after CLAUDE.md: `airtable-sdk-rules.md` (SDK patterns), `reference/design-lessons.md` (stakeholder prefs, UI patterns, AI field gotchas).

### Company Landing Page

| | |
|---|---|
| **Repo** | `https://github.com/ZachFelsenstein-at/company-landing-page.git` |
| **Base** | Company Homepage |
| **What** | Internal company landing page — categories, news, resources, calendar |

Key doc after CLAUDE.md: `airtable-sdk-rules.md` (same SDK, same gotchas).
