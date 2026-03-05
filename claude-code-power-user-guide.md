# Claude Code Power User Guide

A field-tested playbook for getting maximum utility out of Claude Code. These patterns emerged from months of daily use building real systems — not theory.

---

## 1. CLAUDE.md Is Your Most Important File

Claude Code reads `CLAUDE.md` at the root of your project on every conversation start. This is **shared memory between sessions**. Treat it like infrastructure, not documentation.

**What goes in it:**
- Project structure (directories, what lives where)
- Technical stack and integration patterns
- Known issues and gotchas (especially things that don't work)
- Current status and active work
- Key decisions and why they were made

**What doesn't:**
- Lengthy prose — use tables and bullets
- Anything that changes every session (use memory files for that)
- Duplicated info (link to detailed docs instead)

```
# My Project

## Structure
| Directory | What |
|-----------|------|
| `src/`    | Application code |
| `scripts/`| CLI tools and automation |
| `docs/`   | Reference documentation |

## Known Issues
- API rate limit is 100/min — batch calls in `src/api.ts`
- OAuth token refresh silently fails if clock skew >30s

## Technical Decisions
- Chose Vercel over Cloudflare Workers because of blob storage
- Using CJS not ESM because of test mocking (see gotcha below)
```

### Layered CLAUDE.md

You also get:
- **`~/.claude/CLAUDE.md`** — global instructions across ALL projects (your preferences, communication style, workflow rules)
- **`~/.claude/projects/<path>/memory/MEMORY.md`** — auto-memory that persists across conversations

The global file is where you put things like "always use bun instead of npm" or "I prefer flat ESLint config" or "don't add comments unless the logic isn't self-evident."

---

## 2. Permission Friction Reduction

The biggest drag on Claude Code productivity is clicking "allow" on every Bash command. Two strategies:

### Strategy A: Proactive Broadening

Before a multi-step workflow, add allow patterns to `.claude/settings.local.json`:

```json
{
  "permissions": {
    "allow": [
      "Bash(npm run:*)",
      "Bash(python3:*)",
      "Bash(git status:*)",
      "Bash(git diff:*)",
      "Bash(git log:*)",
      "Bash(curl:*)"
    ]
  }
}
```

Keep destructive ops (git push, rm -rf, force flags) behind prompts. The goal is zero friction for read/build/test, full friction for destroy/publish.

### Strategy B: Script Consolidation

When a workflow stabilizes (same steps every time), wrap it in a script. `Bash(python3:*)` covers any Python script, so `python3 scripts/deploy.py` = zero prompts.

---

## 3. Engineering Practices (Set Up Once, Benefit Forever)

Every project should have these from day one. Tell Claude Code to set them up — it will.

1. **Testing** — Unit tests for API handlers and core logic. Use the language's built-in runner (`node --test`, `python3 -m unittest`). No framework bloat.
2. **Linting** — ESLint flat config (JS) or ruff (Python). Error-focused, not style-nazi.
3. **Pre-commit hook** — `.git/hooks/pre-commit` that lints staged files + runs tests. Fails fast.
4. **Deploy verification** — `scripts/smoke-test.sh` that hits deployed endpoints. Don't hope it works.

Create `scripts/test.sh` as the unified test runner. Claude Code will learn to run it before commits.

---

## 4. Session Hygiene

Context doesn't carry between sessions. Your project's `CLAUDE.md` is the bridge. Keep it accurate.

### When to update project docs:
- After completing any task — update progress immediately
- After discovering something new — add to known issues
- After creating/moving/deleting files — update project structure
- When a conversation has gone long — proactively pause and do a doc sweep
- When wrapping up — always do a final update

### Document what didn't work

This is the highest-leverage habit. When something fails or behaves unexpectedly, write it down. Future sessions will try the same thing and hit the same wall unless you leave a note.

```markdown
## Known Gotchas
- iTunes Search API: 403 IP block after ~45 requests at 1s intervals. Use 2s+ delays.
- Node CJS `const { fn } = require('./mod')` captures refs at import time. To mock: clear require.cache, inject mock, THEN require handlers fresh.
- Vercel Blob default caching is immutable. Set `x-cache-control-max-age: 0` for real-time data.
```

---

## 5. Memory System

Beyond CLAUDE.md, use the auto-memory directory (`~/.claude/projects/<path>/memory/`) for knowledge that spans sessions but is too detailed for CLAUDE.md.

**Structure it by topic, not by time:**

```
memory/
  MEMORY.md          # Index + behavioral rules (loaded into context automatically)
  session-log.md     # Reverse-chronological progress checkpoints
  api-patterns.md    # API integration gotchas
  deploy.md          # Deployment patterns and issues
  people.md          # Collaborators, contacts, relevant context
```

MEMORY.md is always loaded — keep it under ~150 lines. Use it as an index that links to topic files. Topic files can grow without limit.

You can tell Claude "remember this across sessions" and it will write to memory. You can also say "forget X" and it will clean up.

---

## 6. Skills (Reusable Workflows)

Skills are YAML-defined prompts that you invoke with `/skillname`. They turn multi-step workflows into one command.

**Location:** `.claude/skills/` directory

**Example skill** (`.claude/skills/deploy.md`):

```yaml
---
name: deploy
description: Build, test, deploy, and verify
user-invokable: true
---

# Deploy Workflow

1. Run the test suite (`scripts/test.sh`). Stop if anything fails.
2. Build the project.
3. Deploy to production.
4. Run smoke tests (`scripts/smoke-test.sh`).
5. Update CLAUDE.md with deployment status and any issues found.
```

Skills should be **agentic, not procedural**. Say "here's what matters, figure it out" not "step 1, step 2, step 3." Give decision authority, not checklists.

---

## 7. MCP Servers (Extending Capabilities)

MCP (Model Context Protocol) servers give Claude Code access to external services. Configure in `~/.claude.json`.

**High-value MCPs:**
- **Playwright** (`@anthropic-ai/mcp-playwright`) — browser automation, testing, scraping
- **Google Workspace** — Gmail, Calendar, Drive access
- **Home Assistant** — smart home control (if applicable)
- **macOS Automator** (`@steipete/macos-automator-mcp`) — AppleScript/JXA execution

**Caution with Playwright:** DOM snapshots eat context fast (100k+ chars). Use it surgically. Prefer `curl` for simple endpoint checks.

---

## 8. Behavioral Principles

These are the meta-patterns that make everything else work:

### Bias to action over asking
When the obvious next step is clear, just do it. Don't ask permission for low-risk, easily reversible actions.

### AI-native, not keyword-matching
When building anything that routes or classifies, give the LLM a schema and let it decide. Never build brittle pattern matching when you have an LLM available.

### Plan before building
For anything non-trivial, enter plan mode first (`/plan` or ask Claude to plan). A 5-minute plan prevents a 2-hour dead end. Save plans to a `plans/` directory so future sessions can reference them.

### Eliminate decoration
Tell Claude your communication preferences in global CLAUDE.md. Examples:
- "Eliminate filler and soft language"
- "Challenge assumptions"
- "Don't add docstrings or comments to code you didn't change"
- "Don't over-engineer — only make changes directly requested"

### Test before building on top
Each phase must be verified before starting the next. Manual testing isn't enough for regression prevention. Write the test, run the test, then build the next thing.

---

## 9. Multi-Project Coordination

If you have multiple repos that share patterns, create a shared core:

```
~/Projects/
  my-core/          # Shared scripts, conventions, patterns
  project-a/
    core -> ~/Projects/my-core  # Symlink
  project-b/
    core -> ~/Projects/my-core  # Symlink
```

The core repo holds:
- Shared CLI tools
- Convention docs
- Technical patterns (API quirks, tool configs)
- Reusable skill templates

**IP boundary rule:** Technical patterns (how tools work) = share freely. Business logic or domain-specific work product = stays in its repo.

---

## 10. The Forward-Looking Posture

Claude Code isn't just a code assistant — it's infrastructure for a **personal operating system**. The trajectory:

1. **Start with CLAUDE.md** — shared memory across sessions
2. **Add skills** — reusable workflows for things you do repeatedly
3. **Add memory** — persistent knowledge that compounds over time
4. **Add integrations** — MCP servers, CLI tools, APIs
5. **Add automation** — daemons, scheduled tasks, event-driven actions
6. **Add agents** — LLM-driven cycles with their own identity, memory, and tools

Each layer builds on the last. The key insight: **governance, identity, and memory are permanent assets. Orchestration code is disposable.** Build the durable layers well, iterate fast on everything else.

### The Decision Framework for What to Build

For any potential project, evaluate on three axes:
- **Interest:** Does this fascinate you? Does it pull attention naturally?
- **Capability:** Does this develop a new or stronger capability? Does it grow you?
- **Connection:** Does this connect you to others or to meaning?

Two of three minimum. All three = go hard.

---

## Quick Start Checklist

- [ ] Create `~/.claude/CLAUDE.md` with your preferences and communication style
- [ ] Create `CLAUDE.md` in your main project with structure, stack, and known issues
- [ ] Set up `.claude/settings.local.json` with permission patterns for your common commands
- [ ] Create `scripts/test.sh` as your unified test runner
- [ ] Set up a pre-commit hook (lint + test)
- [ ] Create your first skill in `.claude/skills/` for your most repeated workflow
- [ ] Tell Claude to "remember X across sessions" for anything you don't want to repeat
- [ ] Install at least one MCP server that matches your workflow

---

## Common Gotchas

| Issue | Fix |
|-------|-----|
| Claude keeps asking before doing obvious things | Add "bias to action over asking" to global CLAUDE.md |
| Permission prompts every command | Proactive broadening in settings.local.json |
| Context lost between sessions | Update CLAUDE.md religiously, use memory files |
| Claude over-engineers solutions | Add "avoid over-engineering, only make requested changes" to CLAUDE.md |
| Claude adds unnecessary comments/docs | Add "don't add docstrings or comments to code you didn't change" |
| Long conversations lose coherence | Claude auto-compresses context; keep CLAUDE.md current so compressed context doesn't lose critical info |
| Plans get lost | Save to `plans/` directory with descriptive names |
| Tests break silently | Pre-commit hook + `scripts/test.sh` |
| Deploy works locally but fails in prod | `scripts/smoke-test.sh` after every deploy |

---

*This guide is a living document. Update it as you discover new patterns.*
