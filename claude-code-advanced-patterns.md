# What You're Not Thinking About Yet

You built your first block. Here's what separates "using Claude Code" from getting 10x leverage out of it. These patterns took ~30 sessions to learn — most of them the hard way.

---

## 1. CLAUDE.md Is Not a README — It's Shared Memory

You've read it. But are you **writing to it**?

Claude has zero memory between sessions. When you close the tab, everything Claude learned — every gotcha, every field ID, every "oh that approach didn't work" — vanishes. Unless it's in CLAUDE.md.

**The discipline:**
- Finished a task → update CLAUDE.md immediately (checkbox, status, notes)
- Hit a weird bug → add it to Known Issues before you forget
- Created or moved a file → update Project Structure
- Wrapping up for the day → final documentation sweep

**Why this matters for collaboration:** When your teammate opens Claude Code tomorrow, Claude reads CLAUDE.md first. If you documented what you built, what broke, and what's next, their session starts at full speed. If you didn't, they'll re-discover everything you already learned.

**The compaction trap:** During long sessions, Claude compresses old messages to free up context. Details from 30 minutes ago silently disappear. CLAUDE.md survives compaction. Your conversation doesn't. Update it *during* work, not just at the end.

---

## 2. Fresh Sessions > Long Grinds

Your instinct will be to keep going in a long session. Resist it.

After ~45-60 minutes of active work, Claude's context window fills up, old messages get compacted, and quality degrades. Claude starts forgetting decisions you made earlier, re-proposing approaches you already rejected, or losing track of which files it modified.

**When to start fresh:**
- Claude repeats something you already discussed
- Claude proposes a change that contradicts an earlier decision
- You've been going for over an hour on complex work
- After any major milestone (feature complete, PR merged, etc.)

**How to start fresh well:**
1. Update CLAUDE.md with current state
2. `/clear` or open a new session
3. Claude re-reads CLAUDE.md and picks up where you left off

A 5-minute fresh start beats a 20-minute fight with degraded context.

---

## 3. Let Claude Fail

This is counterintuitive. When Claude hits an error — a wrong API call, a bad import path, a field that doesn't exist — your instinct is to jump in and fix it. Don't (at least not immediately).

Claude writes to **auto-memory** (MEMORY.md) when it discovers things. If it tries `getField()`, gets an error, switches to `getFieldIfExists()`, and succeeds — that pattern gets saved. Next session, it won't make that mistake again.

If you always pre-empt the error, Claude never learns it. The first session is slower. Every subsequent session is faster.

**When to intervene:** If Claude is going in circles (same error 3+ times) or about to do something destructive. Otherwise, let it work through it.

---

## 4. How You Ask Determines What You Get

The gap between a bad prompt and a good one is 1 iteration vs. 5.

**Bad:** "Make the cards look better"
**Good:** "The project cards in ProjectsView need more visual hierarchy. The title should be more prominent than the metadata. Look at how BetsView cards handle this — same pattern."

**Bad:** "Fix the bug"
**Good:** "When I click Submit on the standup form, nothing happens. No error in console. The `createRecordAsync` call in the submit handler around line 2400 might not be firing. Check if the permission gate is blocking it — we've seen `hasPermissionToCreateRecords()` silently block in interface extensions before."

**The pattern:**
- What you want (specific)
- Where to look (file, line, component)
- What you've already tried or suspect
- Reference existing code that does something similar

Also: **tell Claude what NOT to do.** "Add a filter dropdown. Don't refactor the existing sort logic. Don't add types to functions you didn't change." Claude tends to "improve" surrounding code unless you tell it not to.

---

## 5. Plan Mode for Anything Non-Trivial

If a task touches more than 2-3 files or has multiple valid approaches, use `/plan` first.

Without plan mode, Claude dives straight into coding. If the approach is wrong, you've burned 10 minutes of context and changes you need to revert. With plan mode, Claude researches the codebase, proposes an approach, and waits for your approval before writing a line of code.

**Use plan mode for:**
- New views or major components
- Anything involving schema changes
- Refactoring that touches multiple files
- Features where you're not sure of the best approach

**Skip plan mode for:**
- Bug fixes where you know the cause
- Small UI tweaks
- Adding a field to an existing component

---

## 6. The Release Loop

Release early and often. Don't batch.

```bash
echo "Added filter dropdown to ProjectsView" | block release
```

After every meaningful change, release. Then refresh the Airtable interface page (Cmd+Shift+R if CDN is stale). Verify it works in the real interface, not just the dev server. The dev server and the released version can behave differently — especially around data source permissions and custom properties.

**The cadence:** Code → release → verify in Airtable → commit → next task. Not: code → code → code → release → find out half of it broke.

---

## 7. Interface Designer Is Half the Battle

A lot of "bugs" aren't code bugs — they're Interface Designer configuration issues.

**Things to check when data seems missing or broken:**
- Is the table added as a data source in the interface page settings?
- Are all needed fields visible (not in "Hidden fields") for that table?
- Is "Allow record editing" / "Allow record creation" enabled for tables you write to?
- Did you add a new table to custom properties but forget to map it in the extension settings panel?

**The hidden fields trap:** When you add a new table to the interface data source, some fields auto-show and others land in "Hidden fields." Fields in Hidden return empty from `safeGetCellValueAsString()`. Always check the Fields popover for each table after any config change.

---

## 8. Git Workflow With Claude

Claude handles git natively. You don't need to switch to a terminal.

**Just say:**
- "Create a branch for the filter feature"
- "Commit what we've done so far"
- "Push and open a PR"
- "What's changed since the last commit?"

**Branch convention:** `yourname/feature-name`

**Claude will:**
- Stage the right files (not `.env`, not `node_modules/`)
- Write a descriptive commit message
- Push and create a PR with a summary

**You should:**
- Review the PR in GitHub before merging — Claude's code is good but not infallible
- Never let Claude push to `main` directly
- Ask Claude to show you the diff before committing if you want to review

---

## 9. The REST API Is Your Power Tool

The Blocks SDK is great for reading and writing records inside an extension. But for everything else — schema inspection, bulk operations, table creation, automation setup — use the Airtable REST API directly.

Claude can make API calls for you:
- "What fields are on the Projects table?" → Claude calls `GET /v0/meta/bases/{baseId}/tables`
- "Create a new table called Milestones with these fields..." → Claude scripts it
- "Rename all fields starting with [OLD] to remove the prefix" → Claude batch-renames via API

Your PAT gives Claude the same access you have. The API is faster and more reliable than clicking through the Airtable UI for bulk operations.

**Gotcha:** The API can't delete fields (404) or update formulas (422). For those, rename to `[DELETE]` prefix and handle in the UI.

---

## 10. Permission Friction Adds Up

Every time Claude asks "Can I run this command?" and you click Allow, you've lost 3-5 seconds and a context switch. Over a session, this adds up to minutes.

**Fix it once:**
- Review your `~/.claude/settings.json` allowlist
- Add any commands you've been approving repeatedly
- `Bash(python3 *)`, `Bash(curl *)`, `Bash(jq *)` are safe to pre-approve
- Keep `rm`, `sudo`, `kill` gated

**Sound hooks** (if you haven't set them up): Claude plays a sound when it needs approval and when it's done. You can work on something else and respond to prompts by ear instead of staring at the terminal.

---

## 11. Collaboration Protocol

When multiple people work on the same codebase with Claude Code:

**Before starting work:**
- `git pull` to get latest
- Read CLAUDE.md — what changed since your last session?
- Check for any new Known Issues or convention changes

**While working:**
- Work on a branch, not main
- Update CLAUDE.md as you go (not just at the end)
- If you discover a new gotcha, add it to Known Issues immediately — your teammate might hit it next

**Before stopping:**
- Update CLAUDE.md with what you built, what's blocked, what's next
- Commit and push your branch
- If you found something that affects `airtable-sdk-rules.md`, update that too

**The goal:** Your teammate's next Claude Code session starts with full context about what you did, what you learned, and what's left. No Slack message needed.

---

## 12. Things Claude Is Surprisingly Good At

You'll naturally use Claude for writing code. But it's also strong at:

- **Schema design** — "I need to track milestones per project. What table structure and field types?" Claude will propose a schema and create it via API.
- **Debugging data** — "Why is the DRI showing as blank for these 3 projects?" Claude will query the records and trace the issue.
- **Code review** — "Review this component for SDK anti-patterns" will catch `getField()` calls, missing null checks, hardcoded field names.
- **Bulk operations** — "Rename all [DELETE] fields back to their original names" runs in seconds via API.
- **Git archaeology** — "What changed in ProjectsView since last week?" reads the git log and summarizes.
- **Generating test data** — "Create 5 test projects with milestones and standups" seeds records via API.

---

## Summary: The Compound Habits

| Habit | Why it compounds |
|-------|-----------------|
| Update CLAUDE.md during work, not just at end | Every session starts smarter |
| Let Claude fail before intervening | Auto-memory improves every future session |
| Fresh sessions over long grinds | Consistent quality > diminishing returns |
| Release after every meaningful change | Catch issues early, keep stakeholders current |
| Use plan mode for non-trivial tasks | Prevents 10-minute wrong-direction detours |
| Be specific in your asks | 1 iteration instead of 5 |
| Check Interface Designer when data is weird | 50% of "bugs" are config issues |
| Document what didn't work | Negative knowledge is the most valuable kind |
