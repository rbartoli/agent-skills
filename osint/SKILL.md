---
name: osint
description: Conduct open-source intelligence (OSINT) investigations using public data. Use when the user asks to investigate a domain, IP, username, email, image, or company from public sources; to map an organisation's attack surface; or to gather public background on an entity. Authorised investigations only — CTF, security research, journalism, defensive work. Not for stalking, doxxing, or targeting individuals without consent.
---

# OSINT Investigation Skill

Structured OSINT workflow for Claude Code. Investigates public information about domains, IPs, people, organisations, and digital assets — using only open sources.

## When to use

- User names a target (domain, company, username, email, image) and asks to "investigate", "look up", "find info on", "map attack surface", or "background check"
- CTF challenges involving reconnaissance
- Security research on authorised targets (bug bounty scope, owned infrastructure)
- Journalism / investigative research on public figures or organisations
- Pre-engagement recon for penetration tests (with written authorisation)

## When NOT to use

- User cannot establish authorisation or legitimate purpose (ask once, then refuse)
- Target is a private individual with no public role and no consent
- Intent involves harassment, stalking, doxxing, or harm
- Requested data is clearly behind legal access controls (not "public")

If any of these apply, refuse clearly and explain why.

## Workflow — four phases

Always work through phases in order. Summarise findings between phases so the user can steer.

### Phase 1 — Scoping

Before any tool runs, confirm:
1. **Target** — exact asset (domain, IP range, username, email, image path)
2. **Purpose** — why this investigation is authorised
3. **Depth** — passive only (no contact with target) vs. active (DNS queries, port scans, etc.)
4. **Output format** — report, timeline, raw findings, IOCs

Active techniques against infrastructure you don't own require explicit authorisation. Default to passive unless told otherwise.

### Phase 2 — Reconnaissance (passive)

Gather baseline from public sources without touching the target's infrastructure.

| Target | Techniques |
|--------|-----------|
| Domain | `whois`, historical WHOIS via viewdns.info, certificate transparency (`crt.sh`), archive.org snapshots |
| IP | `whois`, reverse DNS, passive DNS (SecurityTrails, Mnemonic), Shodan / Censys (passive lookups) |
| Username | Sherlock (`pipx install sherlock-project`), WhatsMyName, Namechk — correlate across platforms |
| Email | haveibeenpwned.com, hunter.io verification, Gravatar hash lookup |
| Image | Exif via `exiftool`, reverse search (Google Lens, Yandex, TinEye), geolocation clues |
| Organisation | LinkedIn public profiles, OpenCorporates, Companies House (UK), SEC EDGAR (US) |

See `references/recon.md` for commands, queries, and API notes.

### Phase 3 — Enumeration (active, authorised only)

Only after Phase 1 authorisation is confirmed.

| Technique | Tool |
|-----------|------|
| Subdomain enumeration | `subfinder`, `amass enum -passive`, `assetfinder` |
| Live host check | `httpx`, `dnsx` |
| Tech fingerprinting | `wappalyzer`, `whatweb`, HTTP headers |
| Port scanning | `nmap` (requires authorisation for owner) |
| Vulnerability scanning | `nuclei` (authorised scopes only) |

See `references/enum.md` for flags, rate-limits, and stealth considerations.

### Phase 4 — Correlation & Reporting

1. Deduplicate findings across sources
2. Build a timeline / graph of relationships (person → email → domain → IP → org)
3. Flag confidence levels — `CONFIRMED` (multiple independent sources), `LIKELY` (single credible source), `UNCONFIRMED` (weak signal)
4. Write a report using this structure:
   - **Target & scope**
   - **Methodology** (what was and wasn't attempted)
   - **Findings** grouped by asset type
   - **Indicators of compromise** (if security context)
   - **Unknowns & next steps**

Never state weak signals as facts. Say "appears to be" when unsure.

## Safety rails

- **Store API keys in `.env`** — never inline in notes or commits
- **Rate-limit all queries** to avoid tripping WAFs or getting IP-banned
- **Tor / VPN** for sensitive recon — Shodan and others log query IPs
- **Scope creep** — if you find something outside authorised scope, stop and ask before continuing
- **Legal** — OSINT is legal in most jurisdictions when data is truly public. Scraping behind login, CFAA violations, and GDPR Article 6 issues exist — consult before publishing

## Reusable prompts

- *"Map the attack surface of <domain>"* — runs Phases 1-3 on infrastructure
- *"Background on <person or company>"* — runs Phase 2 passive only, outputs a dossier
- *"Verify claims in <article URL>"* — cross-references named entities against public records
- *"Correlate <username> across platforms"* — Sherlock + Namechk + custom search

## Progressive disclosure

For tool-specific commands, API authentication, and advanced techniques, load from:

- `references/recon.md` — passive recon playbook
- `references/enum.md` — active enumeration playbook
- `references/tools.md` — setup & API key management for theHarvester, Shodan, Censys, SecurityTrails
- `references/reporting.md` — report templates

Don't load these proactively. Load when the phase or tool is actually needed.
