# OSINT Tools — Setup & API Keys

## Install order (covers 80% of investigations)

```bash
# Package managers
brew install whois exiftool jq nmap   # macOS
sudo apt install whois exiftool jq nmap dnsutils   # Debian/Ubuntu

# Go-based (ProjectDiscovery suite)
go install -v github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest
go install -v github.com/projectdiscovery/httpx/cmd/httpx@latest
go install -v github.com/projectdiscovery/dnsx/cmd/dnsx@latest
go install -v github.com/projectdiscovery/nuclei/v3/cmd/nuclei@latest
go install -v github.com/projectdiscovery/naabu/v2/cmd/naabu@latest
go install github.com/OWASP/Amass/v4/...@master
go install github.com/tomnomnom/assetfinder@latest
go install github.com/ffuf/ffuf/v2@latest

# Python-based
pipx install sherlock-project
pipx install theHarvester

# Nuclei templates
nuclei -update-templates
```

## API keys (store in `~/.config/osint/.env`)

```bash
SHODAN_API_KEY=...          # shodan.io — free tier: 100 queries/month
CENSYS_API_ID=...           # censys.io — free tier: 250 queries/month
CENSYS_API_SECRET=...
SECURITYTRAILS_API_KEY=...  # securitytrails.com — free tier: 50 queries/month
VT_API_KEY=...              # virustotal.com — free tier: 500 req/day, 4 req/min
HUNTER_API_KEY=...          # hunter.io — free tier: 25 searches/month
HIBP_API_KEY=...             # haveibeenpwned.com — $3.95/month minimum
GITHUB_TOKEN=...             # for `gh` CLI + code search
```

Load via `source` or use `direnv`:

```bash
# .envrc
dotenv ~/.config/osint/.env
```

## Configure ProjectDiscovery suite

```yaml
# ~/.config/subfinder/provider-config.yaml
virustotal:
  - $VT_API_KEY
securitytrails:
  - $SECURITYTRAILS_API_KEY
shodan:
  - $SHODAN_API_KEY
censys:
  - $CENSYS_API_ID:$CENSYS_API_SECRET
```

## Tor for sensitive recon

```bash
# Install Tor
brew install tor               # macOS
sudo apt install tor            # Debian

# Route one-off curl through Tor
torsocks curl https://crt.sh/?q=example.com

# Or full proxychains
proxychains4 subfinder -d example.com
```

## VPN considerations

- Dedicated residential IP for OSINT separate from personal browsing
- Avoid free VPNs — they often appear on abuse lists and get blocked
- Mullvad, ProtonVPN, IVPN all accept anonymous payment

## Docker images (pre-built toolkits)

```bash
# Kali Linux — full pentest suite
docker run -it --rm kalilinux/kali-rolling

# BlackArch
docker run -it --rm blackarchlinux/blackarch

# OWASP Amass dedicated
docker run --rm caffix/amass enum -d example.com
```
