# Scope Selection — Edge Cases

Which CLAUDE.md or agents.md file should a rule go in? The main SKILL.md covers the common cases. This file handles the edge cases.

## Global (`~/.claude/CLAUDE.md`)

Use for rules that apply to **everything you do** — coding, research, note-taking, any project. Examples:

- *"Always run `date` before stating today's date in writing."*
- *"Before deleting a file, show its contents first."*
- *"Never commit without running the test suite."*

## Project-local (`./CLAUDE.md`)

Use for rules that apply to **this specific project**. Examples:

- *"In this Next.js 16 app, use Server Components by default."*
- *"This codebase uses `bun`, not `npm` or `pnpm`."*
- *"Tests run via `cargo test --workspace`, not per-crate."*

**Finding the project root:**

1. Walk up from the current working directory until you find a `CLAUDE.md` — use that one
2. If none exists, check for a project marker (`.git`, `package.json`, `Cargo.toml`, `pyproject.toml`) — use that directory
3. If multiple candidates, ask the user

## Monorepo sub-projects

In a monorepo, a rule might apply to just one package. Two choices:

- **Sub-package `CLAUDE.md`**: if the package has its own `CLAUDE.md`, put the rule there
- **Root `CLAUDE.md` with a scope note**: if there's only a root `CLAUDE.md`, add the rule but prefix it with the package name:
  ```markdown
  - **[apps/web] Use Tailwind, not CSS modules, for new components.** (2026-04-14)
  ```

Ask the user which they prefer if it's the first time.

## Worktrees

Worktrees share the root's `CLAUDE.md` via the main repo. Write to the main repo's `CLAUDE.md`, not a copy inside the worktree directory.

## Subagents (`agents.md`)

Subagents live at `~/.claude/agents/<name>.md` (user-level) or in a project's agents directory. Their behaviour is governed by their own file.

**When to target a subagent's file:**

- Rule is about a specific subagent's behaviour: *"The `explore` subagent should never write files, only read."*
- Rule wouldn't apply to main-session work

**How to tell:**

Look at what was being discussed when the correction happened. If the user was correcting something a subagent did (visible in the subagent's task description), target that agent's file. If the user was correcting main-session behaviour, don't touch agents.md.

## Missing files

If the target file doesn't exist yet:

1. Confirm with the user before creating it — unexpected `CLAUDE.md` files in project roots can surprise collaborators
2. Create with minimal structure:
   ```markdown
   # <Project name or "Global">
   
   ## Captured Rules
   
   - <new rule>
   ```
3. For project files, ask if they want it git-tracked (default yes — rules are useful to share with teammates and with your future self).

## Private vs shared rules

Project `CLAUDE.md` gets committed and shared with teammates. If a rule is personal (e.g., *"remind me to eat lunch"*), it should go in global, not project. If unsure, ask: *"Is this rule something your teammates would also want, or just you?"*
