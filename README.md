# Sidecar Setup Guide — Interactive Website Prototype

Interactive tutorial website for **Mac mini M4 + iPad 9th generation (Lightning)** — setting up Sidecar as a headless display solution.

## 🌐 Live Demo

**GitHub Pages:** [https://firhyun.github.io/sidecar-setup-guide/](https://firhyun.github.io/sidecar-setup-guide/)

## Features

- **6 Chapters** — Before You Start, Connect Your iPad, Configure BetterDisplay, Make It Automatic, One-Tap Connection, Go Headless
- **Apple-inspired design** — Clean typography, sidebar navigation, subtle gradients
- **Progress tracking** — Chapter completion with localStorage persistence
- **Copy command** — One-click copy for Terminal/BetterDisplay CLI commands with fallback
- **Troubleshooting & FAQ** — Accordion-style help sections
- **Responsive layout** — Desktop sidebar, tablet, and mobile hamburger menu
- **Admin UI prototype** — Invite/manage access simulation (frontend-only, NOT real authentication)

## Run Locally

No build step or dependencies. Open `index.html` in a browser:

```bash
python3 -m http.server 8000
# Then open http://localhost:8000
```

## Project Structure

```
├── index.html        # Complete self-contained single-page application
├── README.md         # This file
├── SOURCE_CODE.md    # Source code reference for handoff
├── SUPABASE_AUTH.md  # Production auth implementation guide
└── .gitignore        # Git ignore rules
```

## ⚠️ Important — Prototype Limitations

This is a **static website prototype** deployed on GitHub Pages:

- **No real authentication** — The "Manage access" and "Sign-in" modals are UI simulations only. They do not secure content, verify identity, or send invitations.
- **No server-side protection** — All content is publicly accessible on GitHub Pages.
- **No credentials committed** — SSH keys, device details, passwords, and personal information must never be added to this repository.
- **localStorage only** — Progress tracking is per-device/per-browser. Allowed users list is in-memory and resets on refresh.

## 🔒 Production Implementation

For production private access, see [SUPABASE_AUTH.md](SUPABASE_AUTH.md) for a complete Supabase Auth implementation guide covering:

- Magic link / invite-only authentication
- Server-side content protection
- User management and allowlist
- Per-user progress persistence
- Revoke access and audit logging

## Repository Management

### Updating Content

1. Edit `index.html` directly — all HTML, CSS, and JS are in one file
2. Commit and push to `main` branch
3. GitHub Pages will auto-deploy within 1–2 minutes

### Adding Screenshots

Replace the visual placeholders in the code with real screenshots:
1. Create an `assets/` directory
2. Add screenshot images
3. Update the `visual()` function in `index.html` to reference real images

### Branch Strategy

| Branch | Purpose |
|--------|---------|
| `main` | Production — auto-deployed to GitHub Pages |
| `dev` | Development and testing |
| `feature/*` | Feature branches for new content |

---

Based on `Guidebook_Sidecar_Mac_Mini_M4_iPad_9 (1).md` (source version Nov 3, 2026).
