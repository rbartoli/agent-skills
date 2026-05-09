---
name: research
description: Deep research on any topic — spawns parallel agents across web, docs, and code sources, synthesizes findings, stress-tests conclusions, and delivers a structured brief. Use this skill whenever the user says /research, asks "research X", "look into X", "what's the state of X", "compare options for X", "should we use X or Y", or any question that needs more than a single search to answer well. Also trigger when the user is evaluating tools, libraries, services, or approaches and needs a thorough comparison before deciding.
---

# /research

Deep-research any topic by gathering evidence from multiple sources in parallel, synthesizing it, and stress-testing the conclusions before delivering a structured brief.

## When to use

- Evaluating tools, libraries, frameworks, or services
- Comparing approaches or architectures
- Investigating a technology, trend, or concept
- Due diligence before building something
- Any question where a single search won't give a confident answer

## Workflow

### Phase 1 — Clarify

Before researching, make sure you understand what the user actually wants to know. Don't assume — ask if ambiguous.

1. Restate the research question in one sentence
2. Identify 2-4 specific sub-questions that need answering
3. Confirm scope with the user: "I'll research X by looking into A, B, and C. Anything else you want covered?"

If the user's intent is already crystal clear (e.g. "research standing desks under £500"), skip the confirmation and go straight to Phase 2 — don't add friction where there's no ambiguity.

### Phase 2 — Parallel research

Spawn 2-4 research agents simultaneously, each covering a different angle. The goal is breadth — each agent searches independently and reports back.

**Agent assignment strategy:**

Pick agents based on the sub-questions from Phase 1. Common patterns:

| Research type | Agent focus areas |
|---|---|
| Tool/library evaluation | Agent 1: official docs + GitHub (stars, activity, issues). Agent 2: community sentiment (Reddit, HN, blogs). Agent 3: alternatives and comparisons |
| Technology deep-dive | Agent 1: primary sources (papers, official docs). Agent 2: practitioner experience (blogs, talks). Agent 3: limitations and criticisms |
| Buy decision | Agent 1: product specs and pricing. Agent 2: reviews and comparisons. Agent 3: deals, availability, alternatives |
| Concept/trend | Agent 1: authoritative definitions and history. Agent 2: current state and key players. Agent 3: criticisms and counter-arguments |

**Each agent should:**
- Use the right tool for the source type (see Tool Selection below)
- Collect direct quotes and specific data points, not vague summaries
- Note the source URL for every claim
- Flag anything that contradicts other findings

**Tool selection — match tool to source:**

| Source | Tool | Notes |
|---|---|---|
| Library/framework docs | Context7 MCP (`resolve-library-id` → `query-docs`) | Always try this first for any named library or framework |
| Web search (broad) | Exa MCP (`web_search_exa`) or `WebSearch` | Use Exa for technical/developer content; WebSearch for general |
| Specific URL content | `WebFetch` first, then Tavily extract as fallback | WebFetch is fastest; Tavily handles JS-rendered pages |
| GitHub repos | `gh` CLI via Bash (`gh repo view`, `gh search repos`) | Stars, last commit, issue count, contributor count |
| Reddit/HN threads | Tavily extract with `extract_depth: "advanced"` | Better than WebFetch for discussion threads |
| Academic papers | WebFetch on arxiv abstract page | Or Tavily for full text extraction |
| News/articles | WebFetch → Firecrawl scrape → Tavily extract | Escalation ladder |

### Phase 3 — Check-in

After agents return, present a mid-research summary to the user:

```
Here's what I've found so far:

**Key finding 1** — [one sentence + source]
**Key finding 2** — [one sentence + source]
**Surprise/contradiction** — [if any]

Should I dig deeper into any of these, or shift direction?
```

This is the user's chance to steer. If they say "looks good" or "continue", proceed to Phase 4. If they redirect, adjust and run additional targeted searches.

### Phase 4 — Synthesize

Combine all findings into a coherent analysis. This is where you add value beyond what any single search could produce:

- Identify patterns across sources
- Resolve contradictions (explain why sources disagree)
- Separate facts from opinions
- Highlight what's well-established vs. uncertain
- Connect findings to the user's specific context (their stack, constraints, preferences)

### Phase 5 — Stress-test

Before presenting conclusions, actively try to break them:

- Search for counter-arguments and failure cases
- Check if the "winning" option has known dealbreakers
- Look for recency — is the information current or outdated?
- Consider what's missing — what couldn't you find? (absence of evidence matters)
- Ask: "If I'm wrong about this, what would that look like?"

If stress-testing reveals a real problem, revise the synthesis. Don't present conclusions you've already undermined.

### Phase 6 — Deliver

Present the research as a structured brief. Format depends on the research type:

**For comparisons/evaluations:**

```markdown
## Research: [Topic]

### Bottom line
[One paragraph: what I'd recommend and why, with caveats]

### Comparison
| Criterion | Option A | Option B | Option C |
|---|---|---|---|
| ... | ... | ... | ... |

### Key findings
1. **[Finding]** — [Evidence + source]
2. **[Finding]** — [Evidence + source]

### Risks and unknowns
- [What I couldn't verify]
- [Where sources disagreed]

### Sources
- [URL 1] — [what it contributed]
- [URL 2] — [what it contributed]
```

**For deep-dives:**

```markdown
## Research: [Topic]

### Summary
[2-3 paragraphs covering the essentials]

### Key findings
1. **[Finding]** — [Evidence + source]

### What the critics say
- [Counter-arguments or limitations]

### Implications for you
- [How this connects to the user's context]

### Sources
- [URL 1] — [what it contributed]
```

### Phase 7 — Transition to action

If the research naturally leads to implementation:

1. Ask: "Want me to plan an implementation based on this?"
2. If yes, enter Plan mode with the research context already loaded
3. The plan should reference specific findings from the research

If the research is informational only, end with the brief. Don't push toward action if the user just wanted to understand something.

## Important principles

- **Every claim needs a source.** No "it's generally considered" or "many developers prefer" without pointing to where you learned this. If you can't source it, say "based on my training data" explicitly.
- **Recency matters.** Flag when information might be outdated. A 2023 blog post about a library that shipped a major rewrite in 2025 is misleading without context.
- **Don't bury the lead.** The bottom-line recommendation goes first. Supporting evidence follows. The user can stop reading after the first paragraph and still have an answer.
- **Acknowledge uncertainty.** "I couldn't find reliable data on X" is more useful than filling the gap with speculation.
- **Match depth to stakes.** A "which npm package for date formatting" question gets a lighter touch than "should we migrate our database". Scale the effort to the decision's impact.
