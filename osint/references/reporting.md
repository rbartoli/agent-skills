# Phase 4 — Reporting Templates

## Quick dossier (person / public figure)

```markdown
# <Name> — OSINT Dossier

**Date:** YYYY-MM-DD
**Target:** <Name>
**Scope:** Public information only, passive recon.
**Purpose:** <journalism / pre-engagement / background check>

## Identity
- Full name: ...
- Known aliases: ...
- Role / affiliation: ...
- Location (public): ...

## Digital footprint
| Platform | Handle | URL | Confidence |
|----------|--------|-----|------------|
| LinkedIn | ... | ... | CONFIRMED |
| GitHub | ... | ... | CONFIRMED |
| Twitter/X | ... | ... | LIKELY |

## Timeline
- YYYY-MM: <event>
- YYYY-MM: <event>

## Unknowns
- ...

## Sources
1. ...
2. ...
```

## Attack surface report (organisation)

```markdown
# <Org> — Attack Surface Report

**Date:** YYYY-MM-DD
**Scope:** <domains / IPs / cloud accounts in scope>
**Methodology:** Passive recon + authorised active enumeration (Phases 1-3)

## Executive summary
- <1 paragraph — what was found, what matters most>

## Assets discovered
### Domains & subdomains
| Asset | IP | Tech | Notes |
|-------|-----|------|-------|
| ... | ... | ... | ... |

### Exposed services
| Host:Port | Service | Version | Notes |
|-----------|---------|---------|-------|

### Cloud assets
| Provider | Asset | Status |
|----------|-------|--------|

## Notable findings
1. **[HIGH]** <finding> — <impact> — <recommendation>
2. **[MEDIUM]** ...

## Out-of-scope assets observed
- <mention only existence, do not investigate further>

## Methodology log
- Tools used: subfinder, amass, nuclei ...
- Queries / wordlists: ...
- Date range: ...
- Rate limit: ...

## Recommendations
1. ...
2. ...
```

## Investigation verification log

When publishing findings (journalism, bug bounty), keep a verifiable log:

```markdown
## Finding: <one-line summary>

**Confidence:** CONFIRMED / LIKELY / UNCONFIRMED

### Evidence
1. **Source 1** (<type>): <url or citation>
   - Accessed: YYYY-MM-DD
   - Hash/screenshot: <archive.org snapshot url>
   - Relevant excerpt: "..."
2. **Source 2** ...

### Corroboration
- Do sources agree? YES / PARTIAL / NO
- Independent? YES / NO (explain)

### Counter-evidence
- <any source that contradicts the finding>

### Notes
- <reasoning, caveats>
```

## Confidence levels — house rules

- **CONFIRMED** — 2+ independent primary sources OR one source with cryptographic verification (signed cert, DNSSEC, PGP)
- **LIKELY** — 1 credible source + circumstantial corroboration
- **UNCONFIRMED** — single signal, or multiple non-independent sources (e.g. one article quoted everywhere)

Never upgrade confidence without new evidence. Never launder `UNCONFIRMED` → `LIKELY` through restatement.
