# Phase 2 — Passive Reconnaissance Playbook

No contact with target infrastructure. All techniques use third-party archives, registries, and search engines.

## Domain

```bash
# WHOIS — current registration
whois example.com

# Historical WHOIS (no auth)
# → https://viewdns.info/iphistory/?domain=example.com
# → https://whoisfreaks.com (paid tier for history)

# Certificate transparency — all certs ever issued for domain/subdomains
curl -s "https://crt.sh/?q=%25.example.com&output=json" | jq -r '.[].name_value' | sort -u

# Archive.org snapshots
# → https://web.archive.org/web/*/example.com
curl -s "http://web.archive.org/cdx/search/cdx?url=example.com&output=json&limit=20"

# Subdomain enumeration (passive)
subfinder -d example.com -silent
amass enum -passive -d example.com
assetfinder --subs-only example.com
```

## IP address

```bash
# Reverse DNS
dig -x 8.8.8.8 +short
host 8.8.8.8

# WHOIS / ASN
whois 8.8.8.8
whois -h whois.cymru.com " -v 8.8.8.8"   # returns ASN, country, allocated date

# Passive DNS (requires API key, free tiers available)
# SecurityTrails, Mnemonic PassiveDNS, VirusTotal
curl -s "https://api.securitytrails.com/v1/history/example.com/dns/a" -H "APIKEY: $ST_KEY"

# Shodan — passive host info (query != scan)
shodan host 8.8.8.8
shodan search "hostname:example.com"
```

## Username

```bash
# Sherlock — 400+ social platforms
pipx install sherlock-project
sherlock <username> --print-found

# WhatsMyName — faster, more comprehensive
# → https://whatsmyname.app

# Namechk — checks availability across 100+ sites
# → https://namechk.com
```

## Email

```bash
# Have I Been Pwned — breach check (manual, no public API without key)
# → https://haveibeenpwned.com/account/<email>

# hunter.io — verify if email exists + find emails by domain
curl "https://api.hunter.io/v2/email-verifier?email=x@example.com&api_key=$HUNTER_KEY"

# Gravatar — public profile linked to any email (MD5 of lowercased email)
echo -n "user@example.com" | md5sum
# → https://www.gravatar.com/<md5-hash>.json
```

## Image

```bash
# Metadata
exiftool image.jpg

# Reverse search (manual — best results)
# → https://lens.google.com
# → https://yandex.com/images (best for faces / Eastern European sources)
# → https://tineye.com

# Geolocation clues — look for:
# - Street signs, car plates, architecture style
# - EXIF GPS tags (often stripped by social media)
# - Shadow angle + timestamp → SunCalc for approx location
```

## Organisation

- **OpenCorporates** — https://opencorporates.com — cross-jurisdiction company search
- **Companies House (UK)** — https://find-and-update.company-information.service.gov.uk
- **SEC EDGAR (US)** — https://www.sec.gov/edgar/searchedgar/companysearch
- **LinkedIn** — use site-specific Google dorks: `site:linkedin.com/in "company name"`
- **Glassdoor / Indeed** — salaries, reviews, tech stack hints

## Google dorking (baseline operators)

```
site:example.com filetype:pdf
intitle:"index of" site:example.com
inurl:admin site:example.com
"confidential" OR "internal use" site:example.com
ext:sql OR ext:bak OR ext:env site:example.com
```

## Breach data

- **DeHashed** (paid) — combined breach corpus, searchable
- **LeakIX** — free indexed breaches and exposed services
- **Pastebin dorks** — `site:pastebin.com "example.com"`

## Cross-referencing

After gathering from multiple sources, ask:
- Does the same email/username/phone appear across >1 source? (confidence booster)
- Do registration dates line up across WHOIS, LinkedIn, GitHub? (timeline)
- Do subdomains reveal internal naming conventions? (e.g. `dev-api.example.com` → likely prod + staging exist)
