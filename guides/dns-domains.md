# DNS & Custom Domains

A cross-cutting reference for connecting custom domains to any deployment platform.

---

## Table of Contents

- [How DNS Works](#how-dns-works)
- [Common Domain Registrars](#common-domain-registrars)
- [DNS Records by Platform](#dns-records-by-platform)
- [SSL/HTTPS Certificates](#sslhttps-certificates)
- [Apex Domain vs www vs Subdomain](#apex-domain-vs-www-vs-subdomain)
- [Subdomains for Multi-App Setups](#subdomains-for-multi-app-setups)
- [DNS Propagation & Checking Tools](#dns-propagation--checking-tools)
- [Troubleshooting](#troubleshooting)

---

## How DNS Works

The Domain Name System (DNS) translates human-readable domain names (e.g., `myapp.com`) into IP addresses that computers use to route traffic. When a user visits your site, their browser queries DNS servers to resolve the domain to the correct server.

### Record Types

| Record | Purpose | Example Value |
|--------|---------|---------------|
| **A** | Maps a domain to an IPv4 address | `185.199.108.153` |
| **AAAA** | Maps a domain to an IPv6 address | `2606:50c0:8000::153` |
| **CNAME** | Creates an alias pointing one domain to another | `cname.vercel-dns.com` |
| **MX** | Directs email to a mail server | `mail.example.com` (priority 10) |
| **TXT** | Stores arbitrary text (SPF, DKIM, domain verification) | `v=spf1 include:_spf.google.com ~all` |
| **NS** | Delegates a domain to specific name servers | `ns1.cloudflare.com` |

**Key rules:**
- A `CNAME` record cannot coexist with other record types on the same name (the apex domain).
- An `A` record is required for apex domains (`example.com`) since `CNAME` is technically not allowed there (though some providers like Cloudflare support "CNAME flattening").
- `AAAA` records are the IPv6 equivalent of `A` records. Add them when your host provides IPv6 addresses.
- TTL (Time to Live) controls how long DNS resolvers cache a record. Lower TTL (300s) means faster propagation when you change records; higher TTL (3600s+) reduces DNS lookups.

---

## Common Domain Registrars

| Registrar | Notes |
|-----------|-------|
| **Cloudflare Registrar** | At-cost pricing, integrated DNS and CDN, free SSL proxy, CNAME flattening for apex domains. Highly recommended if you use Cloudflare for DNS. |
| **Namecheap** | Affordable, good UI, free WhoisGuard privacy. Popular with developers. |
| **Google Domains** | Clean interface, integrated with Google Workspace. Transferred to Squarespace in 2023 but still operates similarly. |
| **GoDaddy** | Largest registrar. Functional but UI can be cluttered with upsells. |
| **Porkbun** | Low prices, clean interface, growing in popularity among developers. |

**Tip:** You can register your domain at one registrar and point your nameservers to a different DNS provider (e.g., register at Namecheap but use Cloudflare DNS). This is common and gives you the best of both.

---

## DNS Records by Platform

### GitHub Pages

GitHub Pages uses four A records for the apex domain and a CNAME for `www`.

**Apex domain (`example.com`):**

| Type | Name | Value |
|------|------|-------|
| A | `@` | `185.199.108.153` |
| A | `@` | `185.199.109.153` |
| A | `@` | `185.199.110.153` |
| A | `@` | `185.199.111.153` |

**www subdomain:**

| Type | Name | Value |
|------|------|-------|
| CNAME | `www` | `<username>.github.io` |

Then add your domain in the repository settings under **Pages > Custom domain**. GitHub will automatically provision a Let's Encrypt SSL certificate (may take up to 24 hours).

---

### Vercel

Vercel uses a single CNAME record. For apex domains, Vercel supports A records as well.

**Subdomain or www (`www.example.com`):**

| Type | Name | Value |
|------|------|-------|
| CNAME | `www` | `cname.vercel-dns.com` |

**Apex domain (`example.com`):**

| Type | Name | Value |
|------|------|-------|
| A | `@` | `76.76.21.21` |

Add the domain via the Vercel dashboard (Project Settings > Domains) or CLI:

```bash
vercel domains add example.com
```

SSL is provisioned automatically within minutes.

---

### Render

Render uses a CNAME pointing to your service's `.onrender.com` subdomain.

| Type | Name | Value |
|------|------|-------|
| CNAME | `www` | `your-service.onrender.com` |

For apex domains, Render provides specific instructions depending on your DNS provider. If your provider supports CNAME flattening (e.g., Cloudflare), you can CNAME the apex directly.

Add the domain in your Render service's **Settings > Custom Domains** section. SSL certificates are provisioned automatically via Let's Encrypt.

---

### Netlify

Netlify supports both CNAME records and their special NETLIFY record type.

**Option 1 - CNAME (for subdomains):**

| Type | Name | Value |
|------|------|-------|
| CNAME | `www` | `your-site.netlify.app` |

**Option 2 - Netlify DNS (recommended for apex):**

If you use Netlify as your DNS provider, they handle apex domains automatically with their `NETLIFY` record type. Point your nameservers to Netlify:

```
dns1.p05.nsone.net
dns2.p05.nsone.net
dns3.p05.nsone.net
dns4.p05.nsone.net
```

**Option 3 - A record for apex:**

| Type | Name | Value |
|------|------|-------|
| A | `@` | `75.2.60.5` |

SSL is automatic via Let's Encrypt.

---

### Railway

Railway uses a CNAME record.

| Type | Name | Value |
|------|------|-------|
| CNAME | `www` | Provided in Railway dashboard |

Add your custom domain in the Railway service settings. Railway generates a unique CNAME target for each service. SSL is automatic.

For apex domains, use a DNS provider that supports CNAME flattening, or add an A record if Railway provides one in the dashboard.

---

### Fly.io

Fly.io supports both CNAME and A records.

**CNAME (for subdomains):**

| Type | Name | Value |
|------|------|-------|
| CNAME | `www` | `your-app.fly.dev` |

**A record (for apex):**

```bash
# Get your app's IP address
fly ips list
```

| Type | Name | Value |
|------|------|-------|
| A | `@` | `<your-app-ipv4>` |
| AAAA | `@` | `<your-app-ipv6>` |

Add the domain and request a certificate:

```bash
fly certs add example.com
fly certs show example.com   # check certificate status
```

---

### Cloudflare Pages

If your domain already uses Cloudflare DNS, the setup is nearly automatic.

1. Go to **Cloudflare Pages > your project > Custom domains**.
2. Add your domain (apex or subdomain).
3. Cloudflare automatically creates the DNS record for you.

If your domain is NOT on Cloudflare DNS, you can still add a CNAME:

| Type | Name | Value |
|------|------|-------|
| CNAME | `www` | `your-project.pages.dev` |

SSL is automatic and handled by Cloudflare's edge network.

---

## SSL/HTTPS Certificates

All modern platforms provision SSL certificates automatically. Here is how each handles it:

| Platform | SSL Provider | Provisioning Time | Notes |
|----------|-------------|-------------------|-------|
| GitHub Pages | Let's Encrypt | Up to 24 hours | Check "Enforce HTTPS" in repo settings |
| Vercel | Let's Encrypt | Minutes | Automatic, no action needed |
| Render | Let's Encrypt | Minutes | Automatic after DNS verification |
| Netlify | Let's Encrypt | Minutes | Automatic, supports wildcard with Netlify DNS |
| Railway | Let's Encrypt | Minutes | Automatic |
| Fly.io | Let's Encrypt | Minutes | Run `fly certs add <domain>` |
| Cloudflare Pages | Cloudflare | Instant | Full or flexible SSL modes available |

**Important:** If using Cloudflare as a DNS proxy (orange cloud), set the SSL mode to **Full (Strict)** to avoid redirect loops. The "Flexible" mode can cause infinite redirect loops with backends that also enforce HTTPS.

---

## Apex Domain vs www vs Subdomain

| Type | Example | Pros | Cons |
|------|---------|------|------|
| **Apex** | `example.com` | Clean, short URL | Cannot use CNAME (needs A record or flattening) |
| **www** | `www.example.com` | CNAME-compatible, CDN-friendly | Slightly longer URL |
| **Subdomain** | `app.example.com` | Isolates services, easy to manage | Extra DNS record per service |

**Best practice:** Pick either apex or www as your canonical domain and redirect the other to it. Most platforms handle this automatically when you add both.

**Recommendation:** If using Cloudflare DNS, use the apex domain (they support CNAME flattening). Otherwise, `www` is technically simpler since it works with standard CNAME records everywhere.

---

## Subdomains for Multi-App Setups

A common pattern for production applications is to use subdomains to separate services:

```
example.com          -> Marketing site (Netlify or Vercel)
app.example.com      -> Web application (Vercel or Railway)
api.example.com      -> Backend API (Render, Railway, or Fly.io)
docs.example.com     -> Documentation (GitHub Pages or Netlify)
staging.example.com  -> Staging environment (same platform, different service)
admin.example.com    -> Admin dashboard (Vercel or Railway)
```

Each subdomain gets its own DNS record pointing to the appropriate platform:

```
app     CNAME   cname.vercel-dns.com
api     CNAME   api-service.onrender.com
docs    CNAME   username.github.io
staging CNAME   staging-app.railway.app
```

**CORS considerations:** When your frontend (`app.example.com`) calls your API (`api.example.com`), you need to configure CORS headers on the API:

```
CORS_ORIGIN=https://app.example.com
```

---

## DNS Propagation & Checking Tools

DNS changes do not take effect instantly. Propagation typically takes **5 minutes to 48 hours**, depending on TTL values and global resolver caches.

### Command-Line Tools

**dig (Linux/macOS):**

```bash
# Query A records
dig example.com A

# Query CNAME records
dig www.example.com CNAME

# Query specific DNS server (Google)
dig @8.8.8.8 example.com A

# Short output
dig +short example.com A
```

**nslookup (all platforms, including Windows):**

```bash
# Basic lookup
nslookup example.com

# Query specific DNS server
nslookup example.com 8.8.8.8

# Query specific record type
nslookup -type=CNAME www.example.com
```

### Online Tools

| Tool | URL | Use Case |
|------|-----|----------|
| **whatsmydns.net** | https://whatsmydns.net | Check propagation globally |
| **DNS Checker** | https://dnschecker.org | Similar global propagation check |
| **MX Toolbox** | https://mxtoolbox.com | Comprehensive DNS diagnostics |
| **Google DNS** | https://dns.google | Check what Google resolvers see |

### Tips for Faster Propagation

1. **Lower TTL before changes:** Set TTL to 300 (5 minutes) a day before making DNS changes. After propagation is complete, raise it back to 3600+.
2. **Flush local cache:** After changing records, flush your local DNS cache:
   ```bash
   # macOS
   sudo dscacheutil -flushcache && sudo killall -HUP mDNSResponder

   # Windows
   ipconfig /flushdns

   # Linux (systemd-resolved)
   sudo systemd-resolve --flush-caches
   ```

---

## Troubleshooting

### DNS Not Propagating

**Symptoms:** Site unreachable, old IP still resolving, "DNS_PROBE_FINISHED_NXDOMAIN".

**Fixes:**
1. Verify records are correct with `dig` or whatsmydns.net.
2. Confirm you edited the correct DNS zone (some registrars have separate DNS management).
3. Wait. Some ISPs cache aggressively. TTL of 3600 = up to 1 hour.
4. Flush your local DNS cache (see above).
5. Try from a different network or device (or use `dig @8.8.8.8` to check Google's DNS).

### SSL Certificate Pending

**Symptoms:** "Your connection is not private," certificate errors, HTTPS not working.

**Fixes:**
1. Confirm DNS records are correctly pointing to the platform.
2. Most platforms require DNS verification before issuing a certificate. Check the platform dashboard for status.
3. GitHub Pages: Uncheck and re-check "Enforce HTTPS" in settings. Wait up to 24 hours.
4. Fly.io: Run `fly certs check example.com` to see certificate status.
5. If using Cloudflare proxy, ensure SSL mode is **Full (Strict)**, not **Flexible**.

### Mixed Content Warnings

**Symptoms:** Page loads but some resources (images, scripts) are blocked. Browser shows "mixed content" warning.

**Fixes:**
1. Ensure all resource URLs use `https://` instead of `http://`.
2. Use protocol-relative URLs (`//example.com/image.png`) or just relative paths (`/image.png`).
3. Add a Content Security Policy header: `Content-Security-Policy: upgrade-insecure-requests`.
4. Search your codebase for hardcoded `http://` URLs and replace them.

### Redirect Loops (ERR_TOO_MANY_REDIRECTS)

**Symptoms:** Browser shows "This page isn't working, redirected you too many times."

**Fixes:**
1. **Cloudflare + platform HTTPS:** Set Cloudflare SSL to **Full (Strict)**, not Flexible. Flexible causes: Browser -> HTTPS -> Cloudflare -> HTTP -> Server -> Redirect to HTTPS -> Cloudflare -> HTTP -> Server -> loop.
2. **Double redirect:** Both `www` and apex redirect to each other. Set one as canonical.
3. **Force HTTPS conflict:** Platform forces HTTPS, and your app also redirects to HTTPS, but the platform terminates SSL at the load balancer and forwards HTTP internally. Check `X-Forwarded-Proto` header instead of the request protocol.

### Domain Verification Failing

**Symptoms:** Platform says "Domain verification failed" or "DNS records not found."

**Fixes:**
1. Ensure you added the records to the correct domain/subdomain.
2. Check for typos in CNAME targets (e.g., `cname.vercel-dns.com` not `cname.vercel-dns.com.` with trailing dot, though some providers add this automatically).
3. Some platforms require a TXT record for verification. Check their docs.
4. If using Cloudflare proxy (orange cloud), temporarily switch to DNS-only (grey cloud) during initial setup, then re-enable proxy after verification.

---

## Quick Reference: Platform Domain Cheat Sheet

| Platform | Apex Domain | www / Subdomain | Auto SSL | Dashboard Path |
|----------|-------------|-----------------|----------|----------------|
| GitHub Pages | 4 A records | CNAME to `user.github.io` | Yes (Let's Encrypt) | Repo Settings > Pages |
| Vercel | A to `76.76.21.21` | CNAME to `cname.vercel-dns.com` | Yes | Project > Settings > Domains |
| Render | Provider-dependent | CNAME to `service.onrender.com` | Yes | Service > Settings > Custom Domains |
| Netlify | A to `75.2.60.5` or Netlify DNS | CNAME to `site.netlify.app` | Yes | Site > Domain management |
| Railway | CNAME flattening | CNAME (from dashboard) | Yes | Service > Settings > Domains |
| Fly.io | A (from `fly ips list`) | CNAME to `app.fly.dev` | Yes (`fly certs`) | CLI or dashboard |
| Cloudflare Pages | Automatic (CF DNS) | CNAME to `project.pages.dev` | Yes | Pages > Custom domains |
