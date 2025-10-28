# Homelab SSO — Authentik + Nginx Proxy Manager + Internal CA

[![Status](https://img.shields.io/badge/status-active-success)]()
[![Made with ❤️ in Chicago](https://img.shields.io/badge/Made%20with%20%E2%9D%A4-Chicago-blue)]()
[![Authentik](https://img.shields.io/badge/IdP-Authentik-6C5CE7)]()
[![NPM](https://img.shields.io/badge/Reverse%20Proxy-Nginx%20Proxy%20Manager-0A84FF)]()
[![License](https://img.shields.io/badge/license-MIT-green)]()

> LAN-only Single Sign-On for homelab services using **Authentik**, **Nginx Proxy Manager**, **ForwardAuth**, **Pi-hole**, and a custom **internal CA** for TLS trust inside a private `.lab` and `.pizza` network.

![Architecture](architecture/sso-overview.png)

---

## 🚀 Elevator Pitch

This project transforms a typical homelab into a cohesive, secure identity platform — mirroring enterprise-grade SSO patterns inside a local environment.

- **Authentik** (`https://saml.lab` @ 192.168.2.65) — Central IdP  
- **Nginx Proxy Manager (NPM)** (`https://npm.pizza` @ 192.168.2.51) — Entry point  
- **ForwardAuth Guard** — Protects NPM admin via Authentik login  
- **Internal CA** — Wildcard certificates for `*.pizza` and `npm.pizza`  
- **Pi-hole** — DNS for `.pizza` and `.lab` domains  

Everything runs **on-prem**, **SSL-secured**, and **documented for reproducibility**.

---

## 🎯 Why I Built This

I wanted to recreate the same identity and trust challenges found in enterprise infrastructure — but within a self-contained homelab.  
This project taught me how **SSO, DNS, and certificates intersect** in real-world production environments.

Goals:

- ✅ Build a secure federated SSO environment  
- ✅ Automate internal certificate management  
- ✅ Strengthen my understanding of PKI and DNS resolution  
- ✅ Showcase advanced homelab engineering for portfolio and recruiters  

---

## 🧩 Components

| Layer | What | Why |
|-------|------|-----|
| **IdP** | Authentik (`saml.lab`) | Central authentication (SAML/OIDC) |
| **Reverse Proxy** | NPM (`npm.pizza`) | Unified web entry point |
| **Guard** | ForwardAuth | Enforces SSO before access |
| **DNS** | Pi-hole | Local resolver for `.pizza` and `.lab` |
| **PKI** | Internal Root CA | Issues leaf certs with SANs |

---

## ⚙️ Architecture Overview

```plaintext
[ Client ]
   │
   ▼
[ Nginx Proxy Manager ] (https://npm.pizza)
   │
   ├──> Reverse proxy to backend services
   │
   └──> ForwardAuth → [ Authentik Guard ] → [ Authentik IdP (saml.lab) ]
                                     │
                                     └──> Redis + Postgres (tokens, sessions)
```

---

## 🛠️ Quick Start (for reviewers)

This repo contains **examples** (no secrets). Replace placeholders and adapt to your environment.

### 1️⃣ NPM Compose (with /ssl mount)
`npm/docker-compose.yml.example`

### 2️⃣ ForwardAuth Guard on :3000
`npm/guard/docker-compose.yml.example`  
`npm/guard/nginx.conf.example`

### 3️⃣ Authentik Stack
`authentik/docker-compose.yml.example`  
Includes Postgres + Redis + Server + Worker.

### 4️⃣ DNS Overrides (Pi-hole)
```
npm.pizza → 192.168.2.51
saml.lab  → 192.168.2.65
```

### 5️⃣ Cert Requirements
Leaf cert **must include SANs**:
```
DNS: *.pizza
DNS: npm.pizza
```

### 6️⃣ Smoke Tests
```bash
# From NPM host (guard redirect test)
curl -I http://127.0.0.1:3000/ -H 'Host: npm.pizza'

# From any client (with CA installed)
curl -I https://npm.pizza
# Expect: 302 Location: https://saml.lab/outpost.goauthentik.io/start?rd=...
```

---

## 🔐 TLS, SANs, and Internal CA

Root CA lives on the NPM host and signs a leaf for `*.pizza` + `npm.pizza`.

### Client Trust Setup

**Ubuntu**
```bash
sudo cp rootCA.crt.pem /usr/local/share/ca-certificates/
sudo update-ca-certificates
```

**Windows**
- Import `rootCA.crt.pem` into *Trusted Root Certification Authorities*.

### Live Verification
```bash
cd tools/certpeek
go mod tidy && go build -o certpeek
./certpeek npm.pizza:443
```

---

## 🔒 ForwardAuth Concept (Nginx + Authentik)

- NPM forwards unauthenticated requests to `127.0.0.1:3000`.  
- Guard applies `auth_request /ak_auth`.  
- If unauthenticated → 401 → redirects to Authentik `/start?rd=...`.  
- After login, Authentik redirects user back → guard allows access.

See `npm/guard/nginx.conf.example` for reference.

---

## 🧪 Troubleshooting

| Symptom | Fix |
|----------|-----|
| **"self-signed in certificate chain"** | Install internal CA on the client |
| **"no alternative certificate subject name matches 'npm.pizza'"** | Reissue cert with SANs `*.pizza`, `npm.pizza` |
| **"Port already in use"** | `sudo ss -tulpn | grep :3000` — stop conflicting process |
| **"Failed to connect to saml.lab"** | Ensure both `.51` and `.65` use Pi-hole for DNS |

More fixes → `docs/troubleshooting.md`

---

## 🗺️ Roadmap

- [ ] Authentik group-based access for NPM Admin  
- [ ] mTLS between Guard ↔ Authentik  
- [ ] GitHub Action: auto-render `.dot` → `.png`  
- [ ] Add screenshots (`login`, `NPM UI`, `auth flow`)  
- [ ] Extend SSO to Grafana, Nextcloud, Portainer, Heimdall  

---

## 🧠 Lessons Learned

This project became a crash course in **identity, trust, and visibility**.

**Key takeaways:**
- 🔄 DNS configuration consistency is everything.  
- 🔐 TLS trust fails silently if SANs don’t match reality.  
- 🧩 SSO success depends on multiple systems agreeing on session state.  
- 🛠️ Debugging is easier when you can *see* cert chains and network flows.  
- 🚀 Automation matters — but so does documentation.

> 💬 “Good infrastructure is invisible — until you try to remove it.”

---

## 🙌 Credits

Built with ❤️ by **Tim Heverin**.

Thanks to the open-source community — especially the **Authentik**, **Nginx Proxy Manager**, and **Pi-hole** teams — for their incredible work and documentation that made this integration possible.  

Special thanks to the **PowerShell**, **Go**, and **DevOps** communities for shared patterns and inspiration.  

This project reflects not just the tooling, but the mindset —  
**automate everything, document everything, and learn from everything.**

---

⭐ **If you found this project helpful or inspiring, give it a star on [GitHub](https://github.com/dj-3dub/homelab-sso)!**
