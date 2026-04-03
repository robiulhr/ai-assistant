# Tasks

## How to use this file
- `[ ]` — not started
- `[~]` — in progress
- `[x]` — done
- When a task reveals sub-tasks → add them inline below it
- Schema tasks always come at the start of the phase they belong to — not all upfront
- Future phases stay as single-line placeholders until you are ready to build them
- Only go deep on the current phase

---

## Phase 0 — Pre-Build
### Status: In Progress

---

- [ ] **0.1 — Architecture Diagram**

  Draw how all pieces connect — containers, services, database, external APIs, data flow between them.

  **What to cover:**
  - Two Docker containers (OpenClaw, Next.js) and how they talk to each other
  - MongoDB — what connects to it
  - Google Drive — which service talks to it
  - External APIs (Claude, Meta, TikTok, YouTube, Google Drive)
  - WhatsApp message flow: user → OpenClaw → Claude → back to user
  - Web app flow: browser → Next.js → MongoDB

  **How deep to go:**
  One diagram. Boxes and arrows. No implementation details — just connections. Should fit on one page. If it doesn't fit on one page it's too detailed.

  **What NOT to think about:**
  - How each service is coded internally
  - Exact API request formats
  - Error handling flows
  - Database field details

  **You are done when:**
  You can look at the diagram and explain to anyone how data moves through the system in 2 minutes.

---

- [ ] **0.2 — Definition of Done per Phase**

  Write one short paragraph per phase answering: "What must work before this phase is considered complete and I move to the next?"

  **How deep to go:**
  2–4 bullet points per phase. Concrete and testable — "WhatsApp message sent and Claude replies" not "WhatsApp is working".

  **What NOT to think about:**
  - Edge cases
  - Performance benchmarks
  - UI polish

  **You are done when:**
  Each phase has a clear pass/fail test you can do in under 5 minutes.

---

- [ ] **0.3 — Database Schema — Phase 1 only**

  Design MongoDB collections needed for Core Infrastructure only. Do NOT design schema for Social Media Manager yet — that comes at the start of Phase 2.

  **Collections to design now:**
  - `settings` — global config (API keys, thresholds, WhatsApp number)
  - `job_queue` — all background jobs (type, status, payload, created, updated)
  - `drive_accounts` — registered Google Drive accounts (label, email, token, space)
  - `drive_files` — file index (file id, drive account, folder path, app, type, size, status)
  - `whatsapp_messages` — message history (direction, content, timestamp, app context)

  **How deep to go:**
  For each collection: field name, type, required/optional, possible values if limited. That's it.

  **What NOT to think about:**
  - Every possible edge case for each field
  - Fields you might need in the future
  - Optimisation and indexes — add those when query performance is actually a problem
  - Relations to collections that don't exist yet

  **Rule:** If you need to imagine a future scenario to justify a field — skip it. Add it when the code needs it.

  **You are done when:**
  Every collection needed for Phase 1 tasks has fields defined at basic level. Nothing more.

---

- [ ] **0.4 — Git Branching Strategy**

  Decide how code changes are managed. You are solo so keep it simple.

  **Suggested approach:**
  - `main` — always working, never broken
  - `dev` — active development branch, merge to main when a phase is complete and tested
  - Feature branches optional — only if a task is risky and you want isolation

  **How deep to go:**
  3 lines. Decide and write it down. Done.

  **You are done when:**
  You know which branch to work on right now.

---

## Phase 1 — Core Infrastructure
### Status: Not Started

**Before starting Phase 1:** Complete Phase 0 tasks 0.1, 0.2, 0.3, 0.4 first.

---

- [ ] **1.1 — Docker Setup**

  Two containers running with correct resource limits and able to talk to each other.

  **What to build:**
  - `docker-compose.yml` with two services: `openclaw` (port 18888) and `webapp` (port 17777)
  - Resource limits: 4GB RAM total, 2 CPU cores total across both containers
  - Shared network so containers can reach each other by service name
  - MongoDB runs outside Docker (already installed on VPS) — containers connect to host MongoDB

  **How deep to go:**
  Get both containers starting without errors. Verify they can ping each other. Verify resource limits are applied.

  **What NOT to think about:**
  - Production-level Docker hardening
  - Container health checks (add later)
  - CI/CD pipeline

  **You are done when:**
  `docker-compose up` starts both containers, both are reachable on their ports, and you can confirm the resource caps are in place.

---

- [ ] **1.2 — OpenClaw Install + Configure**

  OpenClaw running inside its container with Claude API key set.

  **What to build:**
  - OpenClaw installed and running on port 18888
  - Claude API key configured in OpenClaw settings
  - Basic test: send a message to OpenClaw, verify Claude responds

  **How deep to go:**
  Follow OpenClaw setup docs. Configure the minimum needed to get a response from Claude. Nothing else.

  **What NOT to think about:**
  - Custom Claude system prompts yet
  - Multi-application routing yet
  - Token budgets yet

  **You are done when:**
  You send a test message through OpenClaw and Claude replies.

---

- [ ] **1.3 — WhatsApp Connection**

  WhatsApp connected via OpenClaw and verified working.

  **What to build:**
  - QR scan flow working from web app or terminal
  - WhatsApp session persisted (survives container restart)
  - Send a real WhatsApp message to yourself via the system
  - Receive a reply back from Claude

  **How deep to go:**
  Get a real end-to-end message working. One message in, one reply out.

  **What NOT to think about:**
  - Auto-reconnect handling (add in 1.10)
  - Message routing to different applications yet
  - Notification queuing yet

  **You are done when:**
  You send a WhatsApp message to the system and Claude's reply arrives on your phone.

---

- [ ] **1.4 — Next.js App Skeleton**

  Bare Next.js app running with all application sections stubbed out and basic single-user auth.

  **What to build:**
  - Next.js app running on port 17777
  - Route structure matching all 6 applications (pages exist, even if empty)
  - Single-user auth — simple password or token, no signup flow needed
  - Basic layout: sidebar navigation, header, content area
  - Each application section shows a "coming soon" placeholder

  **How deep to go:**
  Routing works, auth works, layout is in place. No real features yet — just the shell.

  **What NOT to think about:**
  - UI design details
  - Mobile responsiveness (do later)
  - Any actual application features

  **You are done when:**
  You can log in, navigate between all application sections, and the app stays up.

---

- [ ] **1.5 — End-to-End WhatsApp ↔ Claude ↔ Web App Pipeline**

  Verify the full pipeline works together before building features on top of it.

  **What to build:**
  - WhatsApp message → OpenClaw → Claude → response back to WhatsApp
  - Web app can read conversation history from MongoDB
  - Basic message routing: all messages currently go to one default handler

  **How deep to go:**
  Prove the pipeline is solid. Send 10 messages. Check they all appear in the web app. Restart containers and verify session restores.

  **What NOT to think about:**
  - Per-application routing yet
  - Complex Claude prompts yet

  **You are done when:**
  10 WhatsApp messages sent, all replied to by Claude, all visible in web app, session survives restart.

---

- [ ] **1.6 — MongoDB Base Collections**

  Core collections from the Phase 0 schema created and accessible from both containers.

  **What to build:**
  - Create collections: `settings`, `job_queue`, `drive_accounts`, `drive_files`, `whatsapp_messages`
  - Seed `settings` with default values (empty API keys, default thresholds)
  - Verify both Next.js and OpenClaw containers can read/write to MongoDB

  **How deep to go:**
  Collections exist, basic validation on required fields, default seed data in place. No indexes yet — add when queries are slow.

  **What NOT to think about:**
  - Social Media Manager collections (Phase 2)
  - Complex validation rules
  - Database migrations strategy (keep simple for now)

  **You are done when:**
  Both containers can read and write to all five collections without errors.

---

- [ ] **1.7 — Google Drive Service**

  Central Drive service working — OAuth, upload, download, file index updated.

  **What to build:**
  - Google OAuth flow from web app — add a Drive account
  - Upload a test file → verify it appears on Drive in correct folder
  - File record created in `drive_files` collection
  - Download/fetch a file by its Drive file ID
  - Multiple accounts: add two accounts, verify routing goes to account with most space
  - Web app: list all registered Drive accounts with space usage

  **How deep to go:**
  Real upload and download working with real Drive account. Multi-account routing working. File index updated on every upload.

  **What NOT to think about:**
  - Migration between accounts (add later when needed)
  - Every error scenario — handle the happy path first, add error handling when you hit actual errors
  - Offline queue for Drive unavailability (add in 1.10)

  **You are done when:**
  Upload a file from web app → appears on Drive → file index has the record → download it back → works.

---

- [ ] **1.8 — Settings Store**

  Global settings readable and editable from web app.

  **What to build:**
  - Web app settings page: read all global settings from MongoDB
  - Edit and save settings (API keys, alert thresholds, WhatsApp number)
  - Both containers read from the same settings — verify a change in web app is immediately reflected in OpenClaw behaviour

  **How deep to go:**
  Settings page works. Save works. Both containers read from same source.

  **What NOT to think about:**
  - App-specific settings (added per application as they are built)
  - Settings history or rollback

  **You are done when:**
  You change a setting in web app and the change is live in both containers immediately.

---

- [ ] **1.9 — Job Queue**

  Three job types set up and processing correctly.

  **What to build:**
  - Job queue collection already exists (from 1.6)
  - Worker that processes jobs: picks up pending jobs, runs them, marks complete or failed
  - Three job types working:
    - Type 1 — Small continuous tasks (short, run frequently)
    - Type 2 — Multi-step pipeline jobs (checkpointed stages, resumable)
    - Type 3 — Heavy jobs (separate processing lane, does not block Type 1/2)
  - Failed jobs: marked failed with error, alert sent via WhatsApp
  - Web app: job queue status visible (pending, running, completed, failed counts)

  **How deep to go:**
  Write one test job for each type. Verify it runs, completes, and shows in web app. Verify a failed job sends a WhatsApp alert.

  **What NOT to think about:**
  - Every job type you will eventually need — just prove the three lanes work
  - Complex retry logic — simple retry once on failure is enough for now

  **You are done when:**
  One test job of each type runs successfully. One intentionally failed job triggers a WhatsApp alert.

---

- [ ] **1.10 — WhatsApp Notification Service**

  Central notification service handling all outgoing WhatsApp messages from all applications.

  **What to build:**
  - All outgoing WhatsApp messages go through one central service — no application sends directly
  - Message queue: if WhatsApp is disconnected, messages wait and send when reconnected
  - Auto-reconnect: if WhatsApp session drops, system attempts reconnect automatically
  - QR rescan: if auto-reconnect fails, web app shows prominent alert with QR scan option
  - Consistent message formatting across all applications

  **How deep to go:**
  Send a message through the service. Disconnect WhatsApp, send another — verify it queues. Reconnect — verify queued message sends. Simulate session drop and verify web app alert appears.

  **What NOT to think about:**
  - Per-application message templates yet (add as each app is built)
  - Message read receipts

  **You are done when:**
  Message queuing on disconnect works. Auto-reconnect works. Web app alert shows on session drop.

---

- [ ] **1.12 — Image Generation Service (Pippit.ai via Puppeteer)**

  Shared image generation service working end to end.

  **What to build:**
  - Puppeteer script that opens Pippit.ai, logs in, submits a prompt, waits for result, downloads the image
  - Wrap as a Job Queue job (Type 2 — multi-step pipeline)
  - Image saved to server temporarily → uploaded to Google Drive → file index updated → caller notified
  - Web app: basic test UI to trigger a generation and see the result

  **How deep to go:**
  Get one real image generated and saved to Drive. Prove the full pipeline works. No application-specific logic yet.

  **What NOT to think about:**
  - Every image type and field (Social Media Manager specific — handled in Phase 2)
  - Simultaneous request handling — Job Queue already handles queuing
  - Alternative tools — Pippit.ai only for now

  **You are done when:**
  Trigger a test generation → image appears on Google Drive → file index has the record.

---

- [ ] **1.11 — System Health Monitoring**

  RAM, CPU, disk, and Drive space monitored with WhatsApp alerts at thresholds.

  **What to build:**
  - Background job (Type 1) running every 15 minutes: check RAM, CPU, disk usage
  - Check Drive account space usage daily
  - WhatsApp alert at configured thresholds (default: 80% warning, 90% urgent)
  - Web app: system status page showing current usage with visual indicators
  - "Show usage" via WhatsApp: one-message snapshot of all current metrics

  **How deep to go:**
  Real metrics from the server. Real alerts sent. Web app page shows live data.

  **What NOT to think about:**
  - Historical charts (add later)
  - External uptime monitor (UptimeRobot — set up separately, outside this codebase)

  **You are done when:**
  Artificially push a metric over threshold → WhatsApp alert received. Ask "show usage" on WhatsApp → snapshot received.

---

## Phase 2 — Social Media Manager: Feature 1
### Status: Not Started

**Before starting Phase 2:** All Phase 1 tasks must be complete and stable.

**First task of Phase 2:** Design the database schema for Feature 1 (task 2.1) before writing any feature code.

---

- [ ] **2.1 — Database Schema — Feature 1**

  Design MongoDB collections for Social Media Manager Feature 1 only.

  **Collections to design:**
  - `businesses` — business profiles
  - `platform_accounts` — connected social accounts per business
  - `products` — product library
  - `categories` — content category library
  - `assets` — all media assets (images, videos, templates) with Drive file reference
  - `reviews` — client reviews per product
  - `slots` — scheduling slots per platform entity
  - `slot_rules` — conditions and rules per slot
  - `post_queue` — posts waiting to go out
  - `posts` — all posts (draft, scheduled, posted, failed)
  - `special_days` — special days calendar

  **How deep to go:**
  Field name, type, required/optional, possible values if limited, and foreign key references to other collections. No more.

  **Rule:** If you need to imagine a scenario you haven't built yet to justify a field — skip it. Add when code needs it.

  **You are done when:**
  Every collection has basic fields defined. You can start writing code against it without guessing what fields exist.

---

- [ ] **2.2 — Business Profile Management**
  Create, edit, switch between businesses. Web app + WhatsApp.

- [ ] **2.3 — Platform Account Connections**
  Facebook OAuth (Pages + Groups + Instagram), Instagram independent OAuth, TikTok OAuth, YouTube OAuth. Connect, disconnect, list connected accounts.

- [ ] **2.4 — Product Library**
  Create, edit, archive products. All fields from schema. Web app full editor. WhatsApp quick add.

- [ ] **2.5 — Content Category Library**
  Create, edit, archive categories. Web app full editor.

- [ ] **2.6 — Asset Management**
  Upload assets (web app + WhatsApp), Drive sync, file index updated, asset states (New / Used / Recently Used / Archived).

- [ ] **2.7 — Client Reviews Management**
  Add review (web app form + WhatsApp forward), approval queue, approve/reject flow, privacy settings.

- [ ] **2.8 — Image Generation via Pippit.ai**
  Puppeteer integration, all image types working, one image at a time flow, approve/reject/generate another.

- [ ] **2.9 — Brand Identity Overlays**
  Logo and watermark placement on generated and uploaded images. Configurable, off by default.

- [ ] **2.10 — Post Creation Flow (Web App)**
  Full 9-step post creation: media selection, format, platform selection, content writing, platform settings, video handling, branding, scheduling, review and confirm.

- [ ] **2.11 — Post Creation Flow (WhatsApp)**
  Conversational flow for quick posts. YouTube redirects to web app.

- [ ] **2.12 — Entity-Based Scheduling + Slot Builder**
  Each platform entity has own slots. Slot builder in web app. Day pattern, time, per-platform content settings.

- [ ] **2.13 — Condition-Based Slot Rules Engine**
  Rules evaluated top to bottom, first match wins. All condition types working. Default rule fallback.

- [ ] **2.14 — Post Queue Management**
  Queue builds from slots. Queue view in web app. Manual reorder, skip, remove.

- [ ] **2.15 — Post Approval Flow**
  Approval required before posting (configurable). Web app approval queue. WhatsApp approval. Reminder after 2 hours of no response.

- [ ] **2.16 — Facebook Pages Posting**
  All post types: photo, video, reel, story, carousel, link, text. First comment for hashtags.

- [ ] **2.17 — Facebook Groups Posting**
  System-timed posting. Failed post alert.

- [ ] **2.18 — Instagram Posting**
  Feed photo, carousel, feed video, reel, story. Auto-crop offer on ratio mismatch.

- [ ] **2.19 — TikTok Posting**
  Video and photo slideshow. Privacy and interaction settings.

- [ ] **2.20 — YouTube Posting**
  Full metadata required. Shorts auto-detection. Thumbnail required. Never instant.

- [ ] **2.21 — Content Recycling System**
  Recycle asset with fresh caption. Off by default. Configurable at 4 levels. Never recycle exact same post.

- [ ] **2.22 — Special Days Calendar**
  Full-year web app timeline. Colour-coded categories. Bangladesh + Islamic + international days. Add/edit/remove custom days.

- [ ] **2.23 — Festival Profiles + Image Generation Flow**
  Profile stores setup recipe only. Always generates fresh image. 2-day advance reminder via WhatsApp. Planning conversation flow.

- [ ] **2.24 — Post History + Performance Tracking**
  All posted content stored with platform performance data. Per product and per category history views in web app.

---

## Phase 3 — Social Media Manager: Feature 2
### Status: Skipped — plan in detail when Feature 1 is complete and stable

---

## Phase 4 onwards — Remaining Applications
### Status: Skipped — plan in detail when Phase 3 is complete

- Phase 4 — Application 2: AI Content Creator
- Phase 5 — Application 3: Reminder & Calendar
- Phase 6 — Application 4: Language Learning
- Phase 7 — Application 5: Personal Knowledge & Brand Intelligence
- Phase 8 — Application 6: Chinese Product Finder
