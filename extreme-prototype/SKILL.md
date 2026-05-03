---
name: extreme-prototype
description: "Rapid parallel prototyping workflow: take a one-line problem statement and spawn 5-20 competing implementations via parallel subagents, evaluate them for 'feel', keep the winner, discard the rest, iterate within hours. Use this skill whenever the user wants to explore a feature space quickly, compare multiple approaches side-by-side, build something without a PRD, or says things like 'build me 5 versions', 'let's prototype this', 'explore approaches', 'which way should we build this', 'vibe code this', or 'extreme prototype'. Also trigger when the user is stuck choosing between architectures or UI patterns — building all of them is now cheaper than debating."
---

# Extreme Prototyping

Building is cheaper than planning. AI agents make code disposable — treat it like sketching. Instead of debating approaches on a whiteboard, build all of them in parallel and keep the one that *feels* right.

## When to use this

- Feature with unclear UX ("should it be a modal or inline?") — build both, click through, pick
- Architecture decision ("monorepo vs separate services?") — scaffold both, stress-test, pick
- UI/landing page exploration — generate 5-10 visual variants, compare live
- Any moment where debating > 15 minutes would be faster resolved by just building all options

## The workflow

### 1. Capture the problem (30 seconds)

Get a single paragraph or even one sentence from the user. This replaces the PRD.

```
User: "I need a dashboard that shows my crypto portfolio performance vs benchmarks"
```

That's enough. No spec doc. No Figma. No stakeholder alignment meeting.

### 2. Identify axes of variation

Before spawning, decide what to vary across prototypes. Pick 2-3 axes:

- **Architecture**: different tech stacks, different state management, different data flow
- **UX pattern**: different layouts, interactions, information hierarchy
- **Scope**: minimal vs feature-rich vs opinionated
- **Style**: different visual directions, tones, densities

Tell the user what you're varying and why. Example:
> "I'll build 5 variants: 2 chart-heavy dashboards (one with D3, one with Recharts), 2 table-first views (one dense, one card-based), and 1 unconventional approach (terminal-style with sparklines). Each takes a different stance on information density."

### 3. Spawn parallel subagents

Launch N subagents (minimum 3, ideally 5-8, up to 20 for visual/UI work). Each gets:
- The same one-paragraph problem statement
- A specific constraint or direction that makes it unique
- Instructions to produce a **working, runnable** artifact — not a plan, not pseudocode
- A time/complexity budget (keep implementations lean — this is a sketch, not production)

Use `isolation: "worktree"` for each agent so they can't interfere with each other.

**Subagent prompt template:**
```
Build a working prototype for: [problem statement]

Your specific direction: [what makes this variant unique]

Constraints:
- Must be runnable — I will test this by [how you'll evaluate]
- Keep it lean: [time/complexity budget]
- Don't over-engineer: this is disposable. Optimize for "does it feel right" not "is it production-ready"
- Save all output to: [workspace path]
```

### 4. Evaluate for "feel"

Once prototypes land, evaluate them. This is NOT a checklist review — it's a gut check:

- **Run each one** (or present screenshots/output if not interactive)
- **Click through** the critical path
- **Ask the user**: "Which one feels closest to what you want?"
- **Note what works** from runners-up (often you steal a detail from variant 3 into the winner from variant 1)

Present results as a comparison table:
```
| # | Approach | Strength | Weakness | Feel |
|---|----------|----------|----------|------|
| 1 | Chart-heavy D3 | Beautiful animations | Slow load, complex | 7/10 |
| 2 | Recharts simple | Fast, clean | Too basic | 5/10 |
| 3 | Table-first dense | Info-rich | Overwhelming | 6/10 |
| 4 | Card-based | Scannable | Wastes space | 8/10 |
| 5 | Terminal sparklines | Unique, fast | Niche taste | 4/10 |
```

### 5. Discard ruthlessly

Delete all losing prototypes. Don't try to salvage architecture from a rejected approach just because it exists. The only value was *learning what feels wrong*.

Keep: the winner + any specific borrowed details from runners-up.

### 6. Iterate on the winner

Take the winning prototype and refine it. Now you can:
- Add the one good idea from variant 3
- Fix the obvious rough edges
- Push to internal users / the user themselves within hours
- Repeat this entire cycle on sub-problems within the winner

## Key principles

- **Code is disposable.** Prototypes exist to be thrown away. Decouple ego from generated code.
- **Parallel beats serial.** 5 implementations in parallel is faster than 5 sequential "let me try another approach" cycles.
- **Feel over spec.** You cannot determine if a UX is good from a static description. You must interact with it.
- **Failure is signal.** The 4 rejected prototypes taught you what *not* to build. That's worth more than a PRD.
- **Show, don't tell.** If proposing a new feature or UI change, present a working prototype — not a slide deck or description.

## Calibrating scale

| Problem complexity | Variants | Agent model | Time budget per variant |
|---|---|---|---|
| UI component / small feature | 5-10 | haiku | 2-5 min |
| Full page / workflow | 3-5 | sonnet | 5-15 min |
| Architecture / system design | 3-4 | opus | 15-30 min |

## Gotchas

- Don't spend time perfecting individual prototypes — they're sketches, not deliverables. If you find yourself refactoring a prototype, you've lost the plot.
- Don't present prototypes sequentially ("here's #1... now #2..."). Present the comparison table first, then let the user drill into any they want to see.
- Don't forget to actually run/render each prototype before evaluating. "Looks good in code" is not the same as "feels good in use."
- Don't vary too many axes at once — if each prototype differs in 5 ways, you can't isolate what made the winner win.

## What this is NOT

- Not an excuse to skip testing the winner (iterate, dogfood, then harden)
- Not useful when requirements are already crystal clear (just build it)
- Not a substitute for user research (prototypes test execution, not whether the problem is real)
