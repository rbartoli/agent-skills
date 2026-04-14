# agent-skills

A small marketplace of Claude Code skills I use daily. Each skill follows Anthropic's progressive-disclosure pattern: tight `SKILL.md` entry point, deeper reference files loaded on demand.

## Skills in this marketplace

| Skill | What it does |
|-------|--------------|
| **[osint](./osint)** | Structured OSINT investigations using public data — four-phase workflow (scoping → passive recon → active enumeration → reporting) |
| **[podcast-process](./podcast-process)** | Transcribe a podcast URL (pca.st / Spotify / Apple / RSS / YouTube) and extract non-obvious insights, references, and action items |
| **[rule-capture](./rule-capture)** | Capture durable behavioural rules from your corrections into `CLAUDE.md` / `agents.md` so the same mistake doesn't repeat next session |
| **[conversation-harvester](./conversation-harvester)** | Mine past Claude Code session transcripts for insights, decisions, and references you'd otherwise lose |

## Installation

### Option 1 — Claude Code marketplace (recommended)

```
/plugin marketplace add https://github.com/rbartoli/agent-skills
/plugin install osint@rbartoli-agent-skills
/plugin install podcast-process@rbartoli-agent-skills
/plugin install rule-capture@rbartoli-agent-skills
/plugin install conversation-harvester@rbartoli-agent-skills
```

Updates: `/plugin marketplace update rbartoli-agent-skills`.

### Option 2 — Manual install (git clone + symlink)

```bash
git clone https://github.com/rbartoli/agent-skills ~/code/agent-skills
for skill in osint podcast-process rule-capture conversation-harvester; do
  ln -s ~/code/agent-skills/$skill ~/.claude/skills/$skill
done
```

Updates: `cd ~/code/agent-skills && git pull`.

### Option 3 — Install a single skill

```bash
git clone https://github.com/rbartoli/agent-skills /tmp/agent-skills
cp -r /tmp/agent-skills/<skill-name> ~/.claude/skills/<skill-name>
```

Loses automatic updates — prefer Option 1 or 2.

## Design principles

1. **Progressive disclosure** — `SKILL.md` is the entry point (~200 lines max). Detailed playbooks live in `references/` and are loaded only when the relevant phase is reached.
2. **Pushy descriptions** — Claude Code under-triggers by default. Skill descriptions use explicit trigger phrases and "use this whenever" language to fight that.
3. **Explicit when-NOT-to-use** — every skill names edge cases where a different tool is the better fit.
4. **Evals included** — every skill ships with `evals/evals.json` containing positive and negative trigger test cases.
5. **Human review before side effects** — skills that write to user-owned files (like `rule-capture`) diff-and-approve, not silent writes.

## Why a marketplace and not individual repos?

Originally each skill was a separate repo. That made four-way sync overhead for every cross-cutting change (description tuning, README polish, marketplace wiring). A single marketplace:

- One `git push` updates all skills
- Users install selectively via the marketplace API
- Cross-skill patterns (e.g. a shared reference on rule format) can be extracted without creating a new repo
- One place to watch for updates

## Contributing

PRs welcome. For new skills, follow the existing structure:

```
<skill-name>/
├── README.md
├── SKILL.md               ← entry point with pushy description
├── references/
│   └── *.md               ← loaded on demand
└── evals/
    └── evals.json         ← trigger + output tests
```

Also add an entry to `.claude-plugin/marketplace.json`.

## License

MIT — see `LICENSE`.
