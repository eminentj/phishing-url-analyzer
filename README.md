# Phishing URL Analyzer

A phishing URL detection tool that checks links for typosquatting, brand impersonation, suspicious domain structure, and newly-registered domains — with a plain-English explanation for every flag it raises.

Built as a personal project to explore how far AI-assisted coding can take someone with no prior programming background. CLI tool + simple web app, both included.

**Live demo:** _add your Render URL here once deployed, e.g. https://phishing-url-analyzer-xxxx.onrender.com_

---

## What it does

Paste in a URL — one you got in an email, text, or DM — and it returns a risk score (0–100) plus the specific reasons behind it:

```
URL:      http://paypa1-secure-login.tk/verify
Domain:   paypa1-secure-login.tk
Score:    50/100
Verdict:  SUSPICIOUS - review before trusting
Reasons:
  - [10 pts] suspicious_tld: Top-level domain '.tk' is heavily abused in phishing/spam campaigns.
  - [ 5 pts] no_https: No HTTPS -- not damning alone, but combined with other flags it raises risk.
  - [35 pts] brand_lookalike: Domain 'paypa1-secure-login.tk' contains 'paypal', which closely
             resembles the brand 'paypal' (similarity 100%). Possible typosquat.
```

**Checks it runs:**
- Brand impersonation — typosquats (`paypa1.com`) and buried fake subdomains (`accounts.google.com.evil.xyz`)
- Domain structure — raw IP hostnames, excessive subdomains, known URL shorteners, suspicious TLDs
- Domain age — newly-registered domains via live WHOIS lookup (a strong phishing signal)

**What it explicitly does NOT do**, by design: it never visits, scans, or sends requests to the URL's destination. It only analyzes the URL string itself and queries public WHOIS records — the same boundary as looking a domain up by hand, just automated.

## Quick start (CLI)

```bash
git clone https://github.com/eminentj/phishing-url-analyzer.git
cd phishing-url-analyzer
pip install python-whois --break-system-packages   # optional, enables live domain-age checks

python3 analyze.py "https://example.com"
python3 analyze.py --batch test_urls.txt            # check a whole list at once
python3 analyze.py "https://example.com" --skip-whois  # skip the network lookup
```

## Quick start (web app)

```bash
pip install flask gunicorn --break-system-packages
python3 app.py
```
Open `http://localhost:5000`.

See [`DEPLOYMENT.md`](DEPLOYMENT.md) for deploying this publicly (Render, PythonAnywhere, or your own server).

## Project structure

| File | Purpose |
|---|---|
| `url_utils.py` | URL structure checks — IP-as-hostname, TLDs, subdomains, shorteners |
| `lookalike.py` | Typosquat + buried-subdomain brand impersonation detection |
| `known_brands.json` | Curated list of frequently-spoofed brands |
| `domain_age.py` | Live WHOIS-based domain registration age check |
| `scorer.py` | Combines all signals into one weighted score + verdict |
| `analyze.py` | CLI entry point |
| `app.py` | Flask web app (form UI + JSON API), with rate limiting |
| `templates/index.html` | Web UI |
| `test_urls.txt` | Mixed known-bad / known-good test set |

## Honest limitations

This is a personal project, not a production security product. A "low risk" score means none of the current checks were triggered — it is **not** a guarantee of safety. It doesn't inspect page content, verify SSL certificate trust chains, or query reputation/blocklist databases (Google Safe Browsing, VirusTotal, etc.). Use it as one signal among several.

The brand list currently covers about 20 well-known companies — real-world phishing targets a much wider range of brands. Contributions to expand it are welcome.

