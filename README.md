# Homelab SSO (Authentik + NPM + Internal CA)

LAN-only SSO setup:

- **Authentik (IdP):** `https://saml.lab` (192.168.2.65)
- **Nginx Proxy Manager (entry):** `https://npm.pizza` (192.168.2.51)
- **ForwardAuth guard** (nginx) sits in front of NPM admin
- **Internal CA** issues `*.pizza` **with SAN** `npm.pizza`
- **Pi-hole** provides internal DNS for `.pizza` and `.lab`

See the diagram: `architecture/sso-overview.png`.
