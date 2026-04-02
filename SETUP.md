# Setup Guide

Step-by-step infrastructure setup guide.
Written as you build — every command, every config decision, every gotcha documented.
Reusable: follow this guide to replicate the exact same setup in any new project.

For feature descriptions → see `FEATURES.md`
For build progress → see `TASKS.md`

---

## System Requirements

- VPS with Ubuntu (Linux 5.15.0+)
- Minimum 4GB RAM allocated to AI Assistant containers
- 2 of 4 CPU cores allocated to AI Assistant containers
- MongoDB already running on port 27017
- Chromium (snap) installed — required for Puppeteer

---

## Stack

| Component | Technology | Port |
|---|---|---|
| WhatsApp + AI gateway | OpenClaw | 18888 |
| Web application | Next.js | 17777 |
| Database | MongoDB | 27017 (external) |
| File storage | Google Drive | — |
| AI | Claude API (Anthropic) | — |

---

## Environment Variables

All secrets and config live in `.env` at the project root.
Never commit `.env` to git — it is in `.gitignore`.

```env
# Claude API
CLAUDE_API_KEY=

# MongoDB
MONGODB_URI=mongodb://localhost:27017/ai-assistant

# Google Drive OAuth
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=
GOOGLE_REDIRECT_URI=

# Meta (Facebook / Instagram)
META_APP_ID=
META_APP_SECRET=

# TikTok
TIKTOK_CLIENT_KEY=
TIKTOK_CLIENT_SECRET=

# OpenClaw
OPENCLAW_API_KEY=

# Web App
NEXTJS_SECRET=
NEXTJS_PORT=17777

# OpenClaw
OPENCLAW_PORT=18888
```

---

## Step 1 — Docker Setup

*(Filled in during Task 1.1)*

---

## Step 2 — OpenClaw Install + Configure

*(Filled in during Task 1.2)*

---

## Step 3 — WhatsApp Connection

*(Filled in during Task 1.3)*

---

## Step 4 — Next.js App Setup

*(Filled in during Task 1.4)*

---

## Step 5 — MongoDB Collections

*(Filled in during Task 1.6)*

---

## Step 6 — Google Drive OAuth Setup

*(Filled in during Task 1.7)*

---

## Gotchas & Known Issues

*(Add any unexpected problems and their solutions here as you encounter them)*

---

## How to replicate this setup in a new project

1. Copy this file to the new project
2. Follow each step in order
3. Replace all environment variable values with new project credentials
4. Steps marked with project-specific notes will need adjustment

