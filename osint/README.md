# osint

An **OSINT investigation skill** for [Claude Code](https://claude.com/claude-code). Gives Claude a structured, four-phase workflow for open-source intelligence: scoping, passive recon, active enumeration, and correlation/reporting.

Built following [Anthropic's skill standards](https://docs.claude.com/en/docs/claude-code/skills) — progressive disclosure, explicit when-to-use gates, and authorisation-required framing for dual-use techniques.

## What it does

When you ask Claude Code to *"investigate example.com"*, *"map the attack surface of X"*, or *"background on Y"*, this skill activates and guides Claude through:

| Phase | Purpose |
|-------|---------|
| **1. Scoping** | Establish target, purpose, authorisation, output format |
| **2. Passive recon** | Public records: WHOIS, certificate transparency, archives, Shodan passive lookups, username correlation, image exif |
| **3. Active enumeration** *(authorised only)* | Subdomain enumeration, port scanning, tech fingerprinting, vulnerability scanning |
| **4. Correlation & reporting** | Deduplicate, build timeline/graph, assign confidence levels, produce a structured report |

Each phase has its own reference file loaded only when needed (progressive disclosure).

## Installation

Part of [rbartoli/agent-skills](https://github.com/rbartoli/agent-skills) — install the whole marketplace or just this skill. See the root README for options.

## Usage

Just ask naturally:

- *"Run OSINT on example.com"*
- *"Map the attack surface for the bug bounty at acme.test (scope: *.acme.test)"*
- *"Background dossier on <public figure> for a journalism piece"*
- *"Correlate the username @example across platforms"*

Claude will:
1. Confirm scope and authorisation before any active scanning
2. Load the relevant phase playbook
3. Report back after each phase so you can steer

## Structure

```
osint/                        ← clone target
├── README.md                 # this file
├── LICENSE
├── SKILL.md                  # Entry point — workflow & phase overview
└── references/
    ├── recon.md              # Phase 2 — passive recon playbook
    ├── enum.md               # Phase 3 — active enumeration (authorised)
    ├── tools.md              # Install order + API key management
    └── reporting.md          # Phase 4 — report templates + confidence rules
```

## Tools covered

**Passive:** `whois`, `dig`, `exiftool`, `crt.sh`, `archive.org`, Sherlock, hunter.io, Gravatar, Companies House, SEC EDGAR, OpenCorporates, Google dorks.

**Active:** `subfinder`, `amass`, `assetfinder`, `httpx`, `dnsx`, `nmap`, `nuclei`, `ffuf`, `whatweb`, `wappalyzer`, ProjectDiscovery suite.

**APIs:** Shodan, Censys, SecurityTrails, VirusTotal, Have I Been Pwned, hunter.io.

## Ethical use

This skill uses **publicly available data** and is intended for:

- Learning OSINT techniques hands-on
- CTF challenges and security training
- Bug bounty programs and defensive security on owned assets
- Journalism and investigative research on public-facing entities
- General curiosity about how your own digital footprint looks

**Don't use it to** harass, stalk, or dox private individuals. That's the only hard line — everything else is fair game with common sense.

Active scanning (Phase 3) touches infrastructure and can be legally grey in some jurisdictions (CFAA in the US, Computer Misuse Act in the UK). Use CTF ranges (TryHackMe, HackTheBox, VulnHub) for practice, and stay within scope for real engagements. The skill itself just documents techniques — what you run is up to you.

## Why a skill, not a platform?

Unlike OSINT dashboards (Shadowbroker, Spiderfoot, Maltego), this skill doesn't install 60 feeds and a web UI. It's a **guided workflow** — Claude Code runs commands on your machine, you review results, and a reference playbook keeps the process rigorous.

Advantages:
- No new infrastructure — runs in your existing Claude Code terminal
- You control what gets executed — Claude asks, you approve
- Results live alongside your code/notes, not in a separate tool
- Adapts to new tools — edit the reference files, no platform rebuild

## Contributing

PRs welcome. If you add a new tool or technique, update:

1. The appropriate `references/*.md` file
2. The tool list in `README.md`
3. A brief mention in `osint/SKILL.md` if it changes the main workflow

Keep techniques **public-sources-only** and **authorisation-first**.

## License

MIT — see `LICENSE`.
