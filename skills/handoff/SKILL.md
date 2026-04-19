---
name: handoff
description: Create a timestamped handoff document that captures what was done, what's left, known issues, key decisions, and a copy-pasteable resume prompt so work can continue cleanly in a new session or with a different AI agent (Claude Code, Codex, another Claude chat, etc.). Use this skill whenever the user says "handoff", "create a handoff", "handoff this session", "/handoff", "wrap up", "end of session", "pause here", or any variation about saving session state for later. Also trigger proactively near the end of a working session when significant progress has been made and the user signals they're stopping (phrases like "I'm done for today", "let's pick this up tomorrow", "I need to step away"). The output is a markdown file saved at `./handoff/YYYY-MM-DD-HHMM-slug.md` at the repo root, plus an inline summary with the resume prompt block shown in chat for quick copy-paste.
---

# Handoff

A skill for capturing session state into a durable handoff document, so the next session (human, Claude Code, Codex, or any other AI) can resume without re-learning context.

## When to use this skill

Trigger on:

1. **Explicit requests**: "handoff", "create a handoff", "handoff this session", "/handoff", "wrap up", "end of session", "pause here", "save where we left off"
2. **Proactive end-of-session signals**: "I'm done for today", "let's continue tomorrow", "I need to step away", "gonna pick this up later". Trigger when substantial work has happened in the session and the user indicates stopping. When triggering proactively, briefly confirm ("Want me to write a handoff before you go?") rather than silently creating files.

Do NOT trigger for casual "pause" in unrelated contexts (e.g., "pause the video") or when the session has had no real progress worth capturing.

## What the skill produces

Two things, always both:

1. **A file** at `./handoff/YYYY-MM-DD-HHMM-slug.md` relative to the repo root
2. **An inline summary** in the chat response, with the Resume prompt as a fenced code block the user can copy directly into a new chat

The inline summary is NOT optional. The whole point is that the user can either pick up the file later OR paste the resume prompt into a new AI session right now.

## Workflow

### Step 1: Locate the repo root

Find the repo root. In order:

1. Run `git rev-parse --show-toplevel`. If it succeeds, use that.
2. If not in a git repo, ask the user which directory is the project root, or use the current working directory if it's obviously a project folder.

Create `./handoff/` at the repo root if it doesn't exist. This is the folder, not `.handoff/` or `docs/handoff/`.

### Step 2: Gather "what was done"

Reconcile two sources, since neither alone is enough:

**From git** (run these; quietly ignore failures on non-git projects):
- `git log --oneline -20` (recent commits)
- `git status --short` (uncommitted changes)
- `git diff --stat HEAD` (files changed since last commit)
- `git diff --stat main...HEAD` or `git diff --stat master...HEAD` (branch divergence, if on a feature branch)

**From the conversation**: scan the current session for what was actually worked on, decided, attempted, debugged, abandoned. Pay attention to:
- Files created or edited (even if not yet committed)
- Problems solved vs. problems deferred
- User corrections and preference changes (these matter for the next session)
- Tools, libraries, or approaches chosen and why

**Reconcile**: git tells you what changed on disk; the conversation tells you why and what's still only in the user's head. Use both. If git shows changes the conversation didn't discuss, mention them anyway. They matter to the next session. If the conversation discussed plans that didn't reach git, flag them as "planned but not done".

### Step 3: Generate the slug

The slug is a short kebab-case descriptor of the session's main thrust. Keep it 2–5 words. Derive it from the dominant topic of the session, not from every small thing touched.

Good slugs: `auth-flow-refactor`, `d1-migration-setup`, `fix-websocket-telemetry`, `comite-reader-skeleton`
Bad slugs: `stuff-we-did`, `various-changes`, `session-1`, a 12-word novel

### Step 4: Write the file

Use this exact template. Fill every section. If a section genuinely has nothing, write "None" rather than deleting the heading. Consistency matters for scanning old handoffs.

```markdown
# Handoff: <Human-readable session title>

**Date:** YYYY-MM-DD HH:MM (local)
**Repo:** <repo name or path>
**Branch:** <current branch, if git>
**Session agent:** <Claude, Claude Code, Codex, etc. Whatever this session is>

## What was done

- Concrete bullet points. Each one is a thing that is now true that wasn't before.
- Include file paths when relevant: `src/api/release.py` (added `/time-series` route)
- Group related changes; don't list every one-line edit as its own bullet
- If commits were made, reference them: `a3f2b1c` (refactor verdict scoring)

## What's left / next steps

- What should happen next, in rough priority order
- Be specific: "implement X" not "continue work"
- Call out anything that's blocking progress
- If the next step depends on the user deciding something, say so

## Known issues & gotchas

- Bugs discovered but not fixed
- Environment quirks (port conflicts, broken deps, platform-specific issues)
- Things that look wrong but are intentional (explain why)
- Drift or inconsistencies that future-you or another agent might trip on

## Key decisions & reasoning

- Decisions made this session that aren't obvious from the code
- Tradeoffs considered and why the chosen path won
- Approaches tried and abandoned (so next session doesn't re-try them)

## Resume prompt

​```
I'm resuming work on <project> from a previous session. Here's where things stand:

**Context:** <1–2 sentences on the project and what we're building>

**Just finished:**
- <bullet>
- <bullet>

**Next up:**
- <bullet>
- <bullet>

**Watch out for:**
- <known issue or gotcha>

**Key decisions already made:**
- <decision + brief reason>

Full handoff doc: `handoff/<filename>.md`

Please pick up from the next-up list. Ask me before making any decisions that aren't already locked in.
​```
```

### Step 5: Show the inline summary

After writing the file, respond in chat with:

1. A one-line confirmation of where the file was saved
2. The Resume prompt section, as a fenced code block, ready to copy
3. Nothing else. No lengthy recap. The file has that. The chat response is for quick action.

Example chat output shape:

> Saved handoff to `handoff/2026-04-19-1430-auth-flow-refactor.md`.
>
> Copy this into your next session:
>
> ```
> [resume prompt block here]
> ```

## Formatting rules

- No em dashes (`—` or `--`) in any output. Use periods, commas, or parentheses.
- Bullet points should be complete thoughts, not fragments. "Added rate limiting to /api/auth" not "rate limiting".
- Prefer specifics over generalities. File paths, function names, commit hashes, error messages make handoffs useful.
- Keep the resume prompt under ~300 words. It's a primer, not a full doc. Cross-AI handoffs (where the next session is a different agent like Codex) tend to land near the top of this range because they need more locked-decision context; same-agent resumes are usually shorter.
- Use the user's actual project terminology (from the conversation) rather than generic phrasing.

## Edge cases

**No git repo:** Skip the git commands silently, rely on conversation. Note `**Branch:** n/a (no git)` in the file.

**Barely any progress:** If the session genuinely had little substance (a few minutes of chat, no code touched), tell the user directly: "Not much to hand off from this session. Want me to write one anyway, or skip it?" Don't pad a thin handoff with filler.

**Session spanned multiple distinct topics:** Pick the dominant one for the slug. In "What was done", group by topic with sub-bullets so it stays scannable.

**User is handing off mid-debugging:** The "Known issues & gotchas" section becomes the most important one. Capture the current hypothesis, what's been ruled out, and the exact error or symptom. This is gold for the next session.

**Existing handoff from earlier the same day:** That's fine. Timestamps disambiguate. Don't overwrite older files. Always create a new file per handoff invocation, even if the slug is the same. The point is a durable session-by-session record. If the user explicitly says "update the last handoff" or "add to the handoff", then edit the most recent file in place; otherwise, new file every time.

**Dirty branch with uncommitted changes:** Don't refuse to write the handoff, but do surface it. Under "What was done", clearly separate committed work from uncommitted work (e.g., a sub-bullet "Uncommitted:" with the files from `git status --short`). In "What's left", add "Commit or stash the uncommitted changes" as an early next step if the changes look intentional, or "Review and discard uncommitted changes" if they look like scratch. The point is the next session (or next agent) should not be surprised by a dirty tree.

## Why the resume prompt format matters

The resume prompt is the highest-leverage part of this skill. A new AI session starts cold. No context, no memory, no idea what the user cares about. The resume prompt has to do three jobs in under 300 words:

1. **Orient** (what are we working on)
2. **Catch up** (what's the current state)
3. **Direct** (what to do next, what to avoid)

That's why it's structured, not prose. An AI reading a prose summary has to infer the structure; an AI reading `**Next up:**` followed by bullets just acts on them. Keep the structure even when the content is thin.

For cross-agent handoffs (the next session is Codex, a different Claude instance, etc.), the resume prompt should be written in second person ("You are picking up...") and should include an explicit "Locked decisions. Do not revert without asking" section. Same-agent handoffs don't need this. The user can push back directly. Different-agent handoffs need the guardrail baked in.
