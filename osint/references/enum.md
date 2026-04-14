# Phase 3 — Active Enumeration Playbook

**AUTHORISATION REQUIRED.** These techniques contact target infrastructure. Never run without explicit written permission from the asset owner.

## Subdomain enumeration (active)

```bash
# Brute-force with wordlist (DNS queries reach target's NS)
amass enum -active -d example.com -brute -w /usr/share/wordlists/subdomains.txt

# ffuf VHOST discovery
ffuf -u https://example.com -H "Host: FUZZ.example.com" -w subdomains.txt -mc 200,301
```

## Live-host probing

```bash
# From subfinder output → check which hosts respond
subfinder -d example.com -silent | httpx -silent -status-code -title -tech-detect

# DNS resolution
dnsx -l subdomains.txt -a -resp
```

## HTTP fingerprinting

```bash
# WhatWeb — tech stack detection
whatweb -a 3 https://example.com

# Wappalyzer CLI
npx wappalyzer https://example.com

# Headers + cookies
curl -sIL https://example.com | grep -iE 'server|x-powered-by|x-aspnet|set-cookie'
```

## Port scanning (nmap)

```bash
# Top 1000 ports, service detection, OS fingerprint — LOUD
sudo nmap -sS -sV -O -Pn <ip> -oA scan_<ip>

# Slow, low-noise — approximates legit traffic
nmap -T2 --max-rate 50 -p 22,80,443,8080,8443 <ip>

# Script scanning — potentially intrusive, use with care
nmap --script default,safe <ip>
```

## Vulnerability scanning (nuclei)

```bash
# Baseline templates — low noise
nuclei -u https://example.com -t http/technologies/ -t http/misconfiguration/

# Full scan — noisy, authorisation required
nuclei -u https://example.com -severity critical,high,medium
```

## Cloud asset discovery

```bash
# S3 bucket enumeration
cloud_enum -k example -qs

# Azure/GCP
cloud_enum -k example -s azure gcp
```

## Rate-limit & stealth

- Default: 5-10 req/sec max against a target
- Use `--rate-limit`, `-T2`, `--max-rate`
- Rotate source IP via VPN or residential proxy if in-scope contract allows
- Respect `robots.txt` unless scope explicitly overrides

## Stop conditions

Halt the phase and report back to the user if:
- A tool returns unexpected auth/session data (credentials, tokens)
- A scan triggers a WAF / IDS / rate-limit response
- Findings include PII for non-target individuals
- Scope is ambiguous about a newly discovered asset
