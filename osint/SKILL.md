---
name: osint
description: Conduct open-source intelligence (OSINT) investigations using public data. Use this skill whenever the user asks to investigate, look up, research, profile, dig into, or find information about a domain, IP address, username, email, image, company, or public figure — even if they don't explicitly say "OSINT". Also trigger on "map the attack surface", "check my exposure", "background on X", "find out about Y", "who owns this", "what's running on this IP", "correlate this username across platforms", "dox myself to find what's leaked", "reverse-lookup this image", or "what does the internet know about this entity". Educational and research tool using publicly available data.
---

# OSINT Investigation Skill

Structured OSINT workflow for Claude Code. Investigates public information about domains, IPs, people, organisations, and digital assets — using only open sources. Intended for learning, research, CTF challenges, journalism, and defensive work on your own assets.

## When to use

- User names a target (domain, company, username, email, image) and asks to "investigate", "look up", "find info on", "map attack surface", or "background check"
- CTF challenges and security training
- Learning OSINT techniques hands-on
- Background research on companies, public figures, or public-facing organisations
- Defensive work on owned infrastructure
- Journalism / investigative research

## When NOT to use

- Intent is to harass, stalk, or dox a private individual
- Requested data is behind authentication or legal access controls (not genuinely public)
- Target is a private individual AND there is no legitimate reason (e.g., authorised PI work, journalism, due diligence, counterparty check, lost-contact reconnection)

These are ethical red lines, not permission gates — everything else is fair game. Lack of public profile alone is **not** a red line: legitimate investigators routinely target private individuals using only public data. Only refuse when intent is harmful or access would require breaking authentication/law.

## Workflow — four phases

Always work through phases in order. Summarise findings between phases so the user can steer.

### Phase 1 — Scoping

Before any tool runs, confirm:
1. **Target** — exact asset (domain, IP range, username, email, image path)
2. **Depth** — passive only (public records, no contact with target) vs. active (DNS queries, port scans, etc.)
3. **Output format** — report, timeline, raw findings, IOCs
4. **Investigation context** — why the target matters (PI/background check, journalism, due diligence, CTF, etc.) — informs depth and source mix

**Common-name disambiguation.** If the target is a person's name that likely resolves to many people (e.g., "John Smith"), do NOT refuse. Ask for disambiguators before running any search:

- Approximate year of birth or age range
- City / region / country of residence
- Profession, employer, or industry
- Known identifiers: social handle, domain, email, photo
- Relationship or known-from (how the user encountered the target)

Two or three selectors usually collapse ambiguity enough to proceed. If the user can't provide any, report the common-name problem and offer to proceed on whoever ranks highest in public-profile searches (flagging the uncertainty).

Active techniques against infrastructure you don't own can be legally grey in some jurisdictions (CFAA in the US, Computer Misuse Act in the UK, etc.). Default to passive unless working on assets you own, CTF ranges, or authorised engagements (bug bounty scope, pentest).

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

### Phase 3 — Enumeration (active)

Touches target infrastructure. Use on assets you own, CTF ranges, or authorised scopes (bug bounty, pentest).

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
- **Legal context** — OSINT is legal in most jurisdictions when data is truly public. Scraping behind login, CFAA violations, and GDPR Article 6 issues still exist — consider these before publishing or taking action on findings

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
