---
layout: post
title: "Working With Evilginx on premises"
date: 2025-09-24
categories: [phishing, security, opsec, redteam]
tags: [phishing, security, opsec, evilginx, caddy, cloudflare, redirector]
---

## TL;DR:

For red-team / phishing exercises where client trust and OPSEC matter, keep captured client data and primary infrastructure on the customer’s own servers (on-prem). Use cloud assets only for thin redirectors/fronts (Cloudflare, Workers, CDNs). Below: Cloudflare → Caddy (edge reverse proxy, internal TLS) → Evilginx (on-prem inside a Tailnet). Cloudflare firewall rule, Caddyfile, Evilginx command and hardening checklist included.

![](/img/diagram_202509.png)

## Why keep client data on the client’s servers (and cloud only for redirectors)

- **Data ownership & legal footprint.** Storing captured credentials/session tokens on client-owned infrastructure avoids moving sensitive material into third-party cloud accounts, reducing legal risk and evidence sprawl.  
- **Containment & auditability.** If logs / captures remain inside the client environment, it’s easier to scope, audit, and destroy them after the exercise.  
- **Operational security.** Cloud fronting (redirectors) can be churned, scaled, and automated while the sensitive back-end is isolated on a private network (VPN / Tailscale / Headscale).


## High-level architecture (what I run)

1. **Cloudflare (public front / redirectors):** DNS + WAF + Redirection rules. Handles TLS to the public and performs cookie checks / redirects so only valid flows reach the phishing surface. Cloud assets are intentionally thin — nothing sensitive is stored there.  
2. **Caddy on an edge server (client-owned):** Terminates TLS with internal/self-signed certs, removes cloud IOCs, and reverse proxies traffic into the private Tailnet. Caddy logs to local files for traceability.  
3. **Tailnet (Headscale/Tailscale):** Private network connecting the Caddy host and the internal evilginx host; avoids exposing evilginx IPs to the public internet.  
4. **Evilginx (on-prem):** Runs inside the tailnet, receives proxied connections on 443, and performs the AiTM/credential capture. Captured data is stored only on the client servers and is accessible only from inside the tailnet.


## The Cloudflare firewall rule I use (cookie gating)

I gate access to the phishing portal by setting a cookie on a benign landing domain. If the cookie is missing, Cloudflare redirects the request away from the portal, this reduces bot hits and automated scanning.

Redirection Rule in Cloudflare:

```
(http.host eq "portal.iamphishy.com")
and (not http.cookie contains "danito=fd6972f3079dbadb191619fe90a40af6")
and not (http.host eq "landing.iamphishy.com" and http.request.uri.path eq "/favicon.ico")
```

**Explanation**

- If the host is `portal.iamphishy.com` **and** the specific cookie is not present → trigger the redirect action.  
- We exempt the landing domain’s favicon so asset requests won’t trigger the redirect loop.  
- Using cookies for gating reduces automated scanners and lets only flows that reached the benign landing continue.


## The Caddyfile

```caddyfile
# Redirect direct IP access to a harmless site (security measure)
1.2.3.4 {
	redir https://not-harmful.com{uri} permanent
}

landing.iamphishy.com {
	log {
		output file /var/log/caddy/landing_access.log
		format console
	}
	tls internal
	encode gzip
	# Apache/nginx in localhost
	reverse_proxy http://127.0.0.1:8000
}

portal.iamphishy.com {
	log {
		output file /var/log/caddy/portal_access.log
		format console
	}
	tls internal
	encode gzip
	reverse_proxy https://evilginx:443 {
		transport http {
			versions 1.1
			tls_insecure_skip_verify
			tls_server_name portal.iamphishy.com
		}
		header_up Host portal.iamphishy.com
		header_up X-Forwarded-Proto https
	}
}

cdn.iamphishy.com {
	log {
		output file /var/log/caddy/cdn_access.log
		format console
	}
	tls internal
	encode gzip
	reverse_proxy https://evilginx:443 {
		transport http {
			versions 1.1
			tls_insecure_skip_verify
			tls_server_name cdn.iamphishy.com
		}
		header_up Host cdn.iamphishy.com
		header_up X-Forwarded-Proto https
	}
}

login.iamphishy.com {
	log {
		output file /var/log/caddy/login_access.log
		format console
	}
	tls internal
	encode gzip
	reverse_proxy https://evilginx:443 {
		transport http {
			versions 1.1
			tls_insecure_skip_verify
			tls_server_name login.iamphishy.com
		}
		header_up Host login.iamphishy.com
		header_up X-Forwarded-Proto https
	}
}
```

**Notes on this file**

- **IP-based redirection:** The `1.2.3.4` block redirects direct IP access to `not-harmful.com`. This prevents scanners and bots from fingerprinting your infrastructure when they access your server's IP directly instead of using domain names. Replace `1.2.3.4` with your actual server's IP address.
- `tls internal` uses Caddy's internal CA (keeps certs off public Let's Encrypt chain and reduces external IOCs).  
- `tls_insecure_skip_verify` is used because the backend (`evilginx`) runs with self-signed certs inside the tailnet — acceptable if the transport is private and access is controlled.  
- Per-vhost logging gives traceability and helps post-exercise cleanup.


## How I run Evilginx (exact command)

Run Evilginx on the internal node like this:

```bash
./evilginx2 -p ./phishlets -t ./redirectors -developer -debug
```

- `-p ./phishlets` → phishlets directory  
- `-t ./redirectors` → redirectors directory (your redirector scripts)  
- `-developer -debug` → developer/debug flags useful during build/testing (reduce verbosity for production).  

**Important:** Evilginx must be reachable **only** from inside the tailnet; do not publish its IP to public DNS.


## Practical hardening & OPSEC checklist

1. **Never expose evilginx IPs in public DNS.** Use private networking (Tailscale/Headscale) or internal hostnames so public scans don’t fingerprint your infrastructure.  
2. **Keep sensitive data on client servers only.** Redirectors must not store or log captured credentials. Only the on-prem evilginx host stores captures.  
3. **Harden redirectors:** rotate domains, use short TTLs, and deploy multiple ephemeral redirectors (Cloudflare Workers are useful for ephemeral logic).  
4. **WAF / firewall rules for gating:** use cookie checks, IP allowlists, or UA validation to reduce bot/scan noise before traffic reaches your landing.  
5. **Log separation & retention:** keep access logs on Caddy and capture logs on the evilginx host. Define and document retention & secure deletion procedures before the engagement.  
6. **Avoid fingerprints:** don’t use predictable subdomain patterns, identical TLS fingerprints across many domains, or public cert chains that create IOCs.  


## Why this pattern works (short)

- Cloud fronting gives resilience, automation and a human-facing surface that’s easy to change.  
- Caddy + internal TLS acts as an OPSEC buffer and central logging point owned by the client.  
- Tailnet isolates the capture server and prevents public fingerprinting of the backend.  

This split (public redirector / private capture) is standard in professional red-team designs.


## Next posts (planned)

In follow-ups I will publish:

- Redirector code examples (Cloudflare Worker / lightweight redirector snippets).  
- The `.htaccess` tweaks and anti-bot measures used on `landing.iamphishy.com`.  

This post explains the **operational design** of running Evilginx on-premises, it is not a full step-by-step Evilginx setup.
