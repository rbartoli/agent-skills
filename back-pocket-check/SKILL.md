---
name: back-pocket-check
description: Use this skill PROACTIVELY whenever the user shares a new technique, framework, mental model, paper, finding, principle, algorithm, pattern, or concept — especially mid-discussion when they're not asking about it explicitly. Trigger phrases include "I just learned", "found this approach", "this paper", "interesting technique", "new framework", "what if we used", "saw this thing about", "Karpathy says", "Feynman's idea about" — but don't limit yourself to those; fire whenever a substantive new concept enters the conversation. The skill silently scans the user's "mental back pocket" of unsolved problems and surfaces a one-line note ONLY if the new concept might crack one of them. Implements Feynman's "12 favorite problems" technique. Stay silent when nothing matches — the value is the rare surprise, not constant noise.
---

# Back-Pocket Check

Implements Feynman's "12 favorite problems" technique. The user keeps a list of unsolved problems they're carrying; whenever a new technique, framework, or finding enters the conversation, you test it against each problem to see whether it might apply.

The famous quote: *"Every time you hear a new trick or a new result, test it against each of your twelve problems to see whether it helps. Every once in a while there will be a hit, and people say, 'How did he do it? He must be a genius!'"*

## When to engage

Engage whenever a substantive new concept enters the conversation — a technique, framework, mental model, research finding, algorithm, principle, metaphor, or pattern from another domain. The signal isn't the user explicitly asking; the signal is *something new that has the shape of a tool*.

Don't engage on:
- Trivial mentions ("I made tea this morning")
- The user repeating something they already know without a fresh angle
- Domain-restating with no new substance ("React is good for UIs")
- Direct task instructions ("write a function to do X")

When in doubt, engage. Undertriggering is the bigger risk — because of the silent-on-no-match rule below, a false positive costs almost nothing (you read a small file and stay quiet).

## What to do when triggered

1. **Read the back-pocket list** at `/home/riccardo/Sync/Obsidian/RB/3. Resources/My mental back pocket.md`. Format: YAML frontmatter + flat checkbox list (`- [ ] problem name`). If the file doesn't exist or is empty, do nothing this turn and don't try again in this session — there's nothing to match against.

2. **Test the new concept against each problem.** Be skeptical. Most of the time, no problem will plausibly match. That's expected — the technique works because real matches are rare.

3. **If zero matches: stay silent.** Output nothing about the check. Don't say "I scanned your back-pocket and found no matches" — that turns a silent feature into a noisy one. The user will eventually ask why you've never said anything, at which point you can explain. Until then, you're doing your job by *not* speaking.

4. **If one or more plausibly match: surface the strongest one.** One match per trigger, even if multiple apply. Format:

```
📎 Back-pocket: this might apply to "<problem-as-listed>". <One-sentence connection.>
```

Drop this as a quick aside, not a derailment. The user can pick it up or ignore it without losing their train of thought.

## Why "silent on no-match" matters

The Feynman technique's economics depend on rare hits, not constant reports. If you announce every check ("I scanned your back-pocket and found nothing related"), you become noise the user filters out, and the system stops working when a real hit comes through. Treat your silence as a feature: by not speaking when there's nothing to say, you preserve the credibility of every surface.

A useful internal test: *if the user said "yes, try that against problem X," would there be a real next step?* If yes, surface it. If you'd just be hand-waving, stay silent.

## Examples

These show the boundary between "match" and "shared keyword."

- **Surface.** New input: *"researchers found weak entropy in early Bitcoin wallet seeds."* Problem: *"bitcoin private keys discovery."* → Specific mechanism, suggests a search strategy. There's a real next step.

- **Stay silent.** New input: *"Bitcoin reached a new all-time high."* Problem: *"bitcoin private keys discovery."* → Shares the word "bitcoin," nothing else. Hand-waving only.

- **Surface.** New input: *"Karpathy's ratchet loop pattern: agent runs a 5-min experiment, commits on success, reverts on failure."* Problem: *"How to make my crypto bot more robust."* → Operational pattern that maps directly to bot dev.

- **Stay silent.** New input: *"Karpathy had an interesting career arc."* Problem: same as above. → Biography, not technique.

- **Stay silent.** New input: *"I made tea this morning."* Problem: any. → Trivial, not tool-shaped.

## Why this skill exists

The user wants to use Feynman's technique without manually re-checking the list every time something interesting comes up. You're the always-on filter. The user trades a small amount of context (you reading a ~10-line file) for the rare moment when you connect a half-remembered finding to an unsolved problem they've been carrying for months.

## Gotchas

- Don't surface matches based on shared keywords alone. "Bitcoin" appearing in both the input and a problem doesn't make a match — the new concept must offer a *mechanism* or *approach* that could advance the problem.
- Don't announce that you checked and found nothing. The entire value of this skill collapses if it becomes noisy on non-matches.
- Don't match against problems the user is actively working on right now in this session — they already have it in working memory. The value is connecting to problems they've *forgotten* they're carrying.
- Don't surface the same match twice in one session. If you already flagged "this applies to X" earlier, don't repeat it when a related concept comes up.

## Source

Feynman's "twelve favorite problems" approach, popularized via various accounts of his methodology. The user maintains the list at `My mental back pocket.md` and updates it manually.
