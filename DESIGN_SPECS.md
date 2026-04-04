# AI Assistant Platform — Planning Document

## Overview
A unified AI-powered personal assistant platform running on a VPS inside Docker.
Accessible via Telegram (for commands/interaction) and a web app (for UI management).
Single user (personal use). No login required for now.

---

## Core Principles

### 1 — "AI Prepares, Human Approves"
AI is not trusted to make final decisions on its own.
AI does all the heavy work — scanning, filtering, scoring, preparing — then presents
a small number of clear options for the user to approve quickly.

The user's input is always fast and simple: yes / no / later / modify.
No information overload. No missed opportunities. No wasted time.

| Application | Feature | AI Does | User Approves |
|---|---|---|---|
| Social Media Manager | Posting | Prepares posts, builds queue, formats per platform | Quick review before going live |
| Social Media Manager | Image Generation | Generates image based on prompt, saves to asset library | ok / retry / modify |
| Social Media Manager | Competitor Monitoring | Flags viral posts, summarizes activity | Decide if action needed |
| Social Media Manager | Product/Content Finding | Surfaces trending topics and products for content | Pick relevant / not relevant |
| AI Content Creator | Idea Finding | Surfaces ideas from your inputs + own discovery | Pick an idea or give direction |
| AI Content Creator | Viral Research | Analyses why reference videos perform, builds style guide | Approve frame-by-frame before it runs |
| AI Content Creator | Script | Writes script based on research findings | Review and approve before production |
| AI Content Creator | Visuals + Video | Generates images, voiceover, assembles video | Approve each step before next begins |
| AI Content Creator | Posting | Posts with title, description, hashtags | Final approval before publishing |
| Reminder & Calendar | Reminders | Suggests reminders based on patterns | Confirm / adjust time |
| Reminder & Calendar | Calendar | Drafts event from natural language | Confirm before saving |
| Language Learning | Lessons | Prepares daily lesson per active language | Read, mark done |
| Personal Knowledge | Intelligence Briefing | Summarizes RSS into concise daily briefing | Read, ask more if interested |
| Chinese Product Finder | Product Discovery | Scores, filters to top 10-15 picks with landed cost + competition data | Pick interested / not / watch later |
| Chinese Product Finder | Supplier Finding | Finds top suppliers, prepares inquiry message | Draft reviewed and sent manually |

This principle applies to every module built now and in the future.

---

### 2 — Human-Like Conversation, Lowest Possible Token Cost

The Telegram experience should feel like talking to a smart person who knows you well —
not interacting with a command-line tool or a chatbot.

**How this works in practice:**

**Natural language always — no strict commands required**
You never need to memorise exact command syntax. Say what you want in plain language.
The system understands intent, not syntax.
Formal commands still work for those who prefer them but are never required.

**Handles broken and vague information**
You will often remember almost nothing — just a fragment, a colour, a vague feeling.
The system searches across your entire history — watchlist, interested list, recent daily
lists, saved suppliers, conversation history — using whatever fragment you gave it.
It never says "I don't know what you mean." It always tries, confirms, and narrows down.

```
"that blue thing I was looking at"     → searches by colour + recent history
"something cheap from last week"       → price range + time filter
"that eid product"                     → seasonal tag search
"the supplier with good rating"        → supplier list sorted by rating
```

**Context awareness — no need to repeat yourself**
System remembers the last few exchanges. You can refer back naturally:
"tell me more about the third one" — no need to restate the product name.

**Responses are always short by default**
Real people talking to each other are brief. The system gives exactly what is needed
in the fewest words. No restating what you said. No long confirmations. No filler.

```
Wrong: "I have successfully added Supplier A to your shortlist.
        You can view your full shortlist anytime in the web app."
Right: "Added ✓"
```

**Only expands when you ask**
Never volunteers more than needed. Waits to be asked for detail.

**Confirmations are always yes/no or number choice**
When unsure, system picks most likely interpretation and asks one short question:
```
You:    "that fan thing"
System: "Neck Fan with Power Bank?"
You:    "yes"
```

**Token cost kept minimal throughout**
- Claude only receives relevant recent context — not full conversation history
- Static responses (confirmations, lookups, list additions) handled without Claude
- Claude only called when real language understanding or thinking is needed
- One short clarification question, never a long analysis of ambiguity

**The design principle in one sentence:**
Talk like a smart person who knows you well, remembers everything, never wastes
your time, and never shows off how much they know.

---

### 3 — Real Business Actions Are Always Separate and Manual

Any action with real-world consequences — sending a message to a supplier,
posting to social media, making a purchase — is always a separate conscious step.
Never bundled into an automated flow. Never triggered without explicit human intent
at that specific moment.

#### Supplier Inquiry — Three Separate Stages

**Stage 1 — Discovery (automated daily)**
Product surfaces in your daily list. You mark it Interested. Deep investigation runs automatically. Nothing else happens. No supplier search triggered yet.

**Stage 2 — Supplier Research (you initiate when ready)**
Happens when you decide — could be same day, could be days later. You trigger it yourself from Telegram or web app:
- Telegram: say something like "find suppliers for that neck fan" or "!supplier neck fan power bank"
- Web app: open the product and click Find Suppliers

System finds and compares top 5 suppliers. Telegram gets a short summary — name, price, MOQ, rating, factory or trader. Web app shows full comparison table. You shortlist one or two. Nothing sent yet.

**Stage 3 — Inquiry Draft and Send (web app only, fully manual)**
Only from the web app. Never from Telegram.
When you are ready — open the supplier profile, read the AI-drafted inquiry message, edit it however you want, then press Send yourself. System never sends anything without you explicitly pressing send on that exact message at that exact moment.

Draft and send are two different actions even within Stage 3. You can save a draft and come back to it later. Nothing goes anywhere until you press send.

```
Stage 1 — Discovery        → automated, no action needed
Stage 2 — Supplier search  → you trigger (Telegram or web app)
Stage 3 — Draft            → web app only, you read and edit
Stage 3 — Send             → web app only, you press send explicitly
```

---

---

## Infrastructure

- **VPS**: Ubuntu, 4 CPU, 16GB RAM, 108GB disk
- **OpenClaw Port**: 18888
- **Main App Port**: 17777
- **Containers**: 2 (OpenClaw + App)
- **LLM**: Claude API (Anthropic)
- **Deployment**: Single `docker-compose up`, fully portable

---

## Core Infrastructure Plan

### Isolation from Existing Applications

The VPS runs other higher-priority applications — 2 Next.js ecommerce apps and 2 Express APIs (plus testing versions of each). These are completely separate from the AI Assistant and must never be affected by it.

- AI Assistant runs entirely inside Docker containers — isolated from all other processes
- Existing apps run outside Docker as they currently are — no changes made to them
- No port conflicts — ports 18888 and 17777 are confirmed free
- Future plan: consolidate all apps into one management setup — but only after AI Assistant is fully stable, as a separate dedicated project

---

### Resource Limits — AI Assistant Containers

Hard limits set on AI Assistant Docker containers. These limits are enforced by Docker — the AI Assistant physically cannot exceed them regardless of workload. Existing apps always have priority.

| Resource | AI Assistant Hard Limit | Reason |
|---|---|---|
| RAM | 4GB maximum | Worst case all existing apps running with traffic uses ~8GB — 4GB cap leaves safe headroom |
| CPU | 2 of 4 cores maximum | Existing apps always have the other 2 cores available |
| Disk (server) | Temporary workspace only | All files uploaded to Google Drive immediately after job completes, then deleted from server |

When AI Assistant hits its RAM or CPU cap it slows down its own background work. It does not crash. Existing apps are completely unaffected.

---

### Storage Strategy

No permanent files stored on the VPS. Server disk is temporary workspace only.

- Files (images, videos, documents) exist on the server only while a job is actively being processed
- Immediately after job completes — uploaded to Google Drive, deleted from server
- Temporary workspace stays small at all times
- All permanent storage lives on Google Drive

---

### System Management — Approach D + E

**D — Three-layer management:**

- **Telegram** — proactive alerts when something breaks or usage gets high. You know instantly without going to check anything.
- **Web app dashboard** — visual status page showing all services, resource usage, and logs. Accessible from any browser including phone. For the circular problem (if OpenClaw/Telegram itself breaks, it cannot alert you via Telegram) — the web app shows the alert instead since it is a separate container.
- **Terminal (SSH)** — always available as fallback for serious issues and deploying updates. Never the first option but always there.

**Telegram reconnection:** When Telegram session expires or drops, the web app shows a QR code page — you scan from your phone browser to reconnect without needing a computer.

**E — External uptime monitor (UptimeRobot, free):**
An external service outside the VPS pings the app every 5 minutes. If the VPS itself goes down entirely, this sends an alert via email or Telegram. Completely independent — works even when the entire server is offline.

---

### Usage Visibility

You can check resource and API usage at any time.

**From Telegram — quick summary on demand:**
Ask anything like "show usage" or "how is the server" and get a one-message snapshot:
```
Server: RAM 2.1GB/4GB  CPU 34%  Temp disk: 1.2GB
Claude API: 45K tokens today  ~$0.18  Month: $3.40
Drive: 28GB used of quota
Jobs: 3 running  12 queued  47 done today
```

**From web app — full monitoring dashboard:**
- RAM, CPU, disk usage with history charts
- Claude API usage broken down by application and by day
- Google Drive storage breakdown
- Container status and health
- Background job history — running, queued, completed, failed

---

### Proactive Alerts — Two Levels

All alert thresholds are configurable from the web app. Defaults:

| What | Early Warning | Critical Alert |
|---|---|---|
| AI Assistant RAM | 70% of 4GB cap (2.8GB) | 90% of cap (3.6GB) |
| Server RAM overall | 75% of total | 90% of total |
| CPU sustained | AI Assistant above 70% for 10+ min | Above 90% for 10+ min |
| Disk temp workspace | Temp files exceed 3GB | Exceed 6GB |
| Claude API daily spend | Hits your configured daily limit | Hits 90% of monthly budget |
| Google Drive | 70% of quota used | 90% of quota used |

**Early warning example:**
```
⚠️ RAM getting high — AI Assistant at 2.9GB of 4GB cap.
Background jobs slowing down. Check app for details.
```

**Critical alert example:**
```
🔴 AI Assistant RAM at limit. Background work paused.
Existing apps unaffected. Check app to investigate.
```

Claude API daily and monthly budget limits are set by you in the web app settings.

---

### Claude API — Token Efficiency Strategy

The goal is to get the best output from the fewest tokens. This requires a strategy across all applications — not just monitoring spend after the fact.

#### Three Model Tiers

Every task is assigned a tier. Never use a heavier model where a lighter one is enough.

| Tier | Model | Use When |
|---|---|---|
| Heavy | Claude Opus | Complex analysis, final product scoring, script writing, viral research interpretation |
| Standard | Claude Sonnet | Script drafts, summarization, moderate reasoning tasks |
| Light | Claude Haiku | Batch triage, simple classification, short confirmations, keyword matching |

---

#### Per-Application Token Strategy

| Application | Heavy (Opus) | Standard (Sonnet) | Light (Haiku) | Zero Claude |
|---|---|---|---|---|
| Social Media Manager | Growth analysis, competitor insight | Post drafting, caption writing | Short confirmations | Queue management, scheduling logic |
| AI Content Creator | Viral research interpretation, final script | Script drafts, visual planning | Step confirmations | File handling, upload, basic tracking |
| Reminder & Calendar | — | Natural language event parsing | Reminder confirmations | Scheduling, recurring logic |
| Language Learning | — | Lesson generation, notes feedback | Daily lesson delivery | Schedule, reminders |
| Personal Knowledge | Weekly deep analysis | Daily briefing summaries | Category tagging | RSS fetching, text matching |
| Chinese Product Finder | Final product analysis (10-15 products) | Grouped theme analysis | Batch triage (30-50 products) | All platform counting, keyword states, queue |

---

#### Four Rules to Minimise Token Cost

**Rule 1 — Filter before Claude sees anything**
Claude only receives what has already passed human config filters and number-based filters. Raw data never goes to Claude. Applies to every application.

**Rule 2 — Batch, never one at a time**
Never send one item to Claude and wait, then send the next. Group everything together and send in one call. Ten products in one prompt is far cheaper than ten separate prompts.

**Rule 3 — Cache repeated work**
If Claude already answered a question about the same content recently — use the cached answer. Zero cost for cached responses.

Examples of cacheable work:
- Keyword category classification (same keyword → same answer)
- Product category fit check (same product + same category settings)
- RSS article relevance for recurring topics

**Rule 4 — Short focused prompts, no filler**
Every prompt is written to be as short as possible while giving Claude exactly what it needs. No long explanations, no repeated context, no restating what was already said.

---

#### Per-Application Daily Budget Allocation

You set a total monthly budget. The system splits it across applications. Adjustable from web app at any time.

| Application | Default Share | Why |
|---|---|---|
| Chinese Product Finder | 35% | Runs daily background analysis — highest Claude usage |
| AI Content Creator | 30% | Script writing and research are token-heavy |
| Social Media Manager | 15% | Post drafting, competitor analysis |
| Personal Knowledge | 12% | Daily briefing summaries |
| Language Learning | 5% | Lesson generation |
| Reminder & Calendar | 3% | Minimal Claude usage |

When an application approaches its daily allocation — it switches heavier tasks to a lighter model first, then queues non-urgent tasks for the next day rather than overspending.

---

#### Priority When Budget Gets Tight

When total daily budget is running low — tasks are prioritised in this order:

1. Anything you triggered yourself (manual requests always answered)
2. Time-sensitive alerts (urgent signals, breaking news matches)
3. Daily briefings and reports
4. Background scanning and analysis
5. Pre-scheduled batch work

Lower priority tasks queue for tomorrow. You get a Telegram note if significant background work was deferred.

---

#### Claude API Dashboard

**Per application:**
- Tokens used today vs daily allocation
- Cost today, this week, this month
- Which tasks consumed the most tokens
- Model tier breakdown (Opus vs Sonnet vs Haiku split)

**Overall:**
- Total spend today and this month vs your monthly budget
- Daily spend trend chart
- Most expensive tasks across all applications
- Cache hit rate — how much was saved by caching

---

## Shared Services

### 1 — Job Queue

Every application has background tasks. All of them go into one central queue. The queue runs them, retries failures, and tracks the status of every job.

#### Three Types of Work — Each Handled Differently

**Type A — Continuous small tasks (e.g. Chinese Product Finder scanning all day)**
Not one long job — a continuous stream of small tasks, each taking seconds. Search one keyword, check one platform, read one page. They flow through the queue all day naturally. No single task is long enough to block anything else.

**Type B — Multi-step pipeline (e.g. AI Content Creator — research → script → visuals → video → post)**
A pipeline that moves through stages over hours. Each stage completes, saves its result, then the next stage begins. If the system restarts mid-pipeline — it resumes from the last completed stage, not from the beginning. Progress always checkpointed. You can see exactly which stage it is on in real time from the web app.

**Type C — Heavy one-time jobs (e.g. frame-by-frame video analysis)**
Run in a separate lane — they do not touch the main queue at all. The main queue keeps flowing normally while heavy jobs run alongside. Only one heavy job at a time to respect the CPU limit.

| Job Type | How Handled |
|---|---|
| Continuous small tasks | Flow through queue naturally, never block |
| Multi-step pipeline | Checkpointed stages — resume from last saved point if interrupted |
| Heavy one-time jobs | Separate lane — main queue unaffected |

#### General Queue Behaviours
- If a job fails — retry automatically a set number of times, then mark failed and alert you via Telegram
- Priority levels — your manual requests jump the queue ahead of background work
- All jobs from all applications visible in one place in the web app

---

### 2 — Scheduler

For time-based work — "run this at 8am every day", "post this at 7pm", "send weekly report every Sunday". The scheduler triggers tasks at specific times and hands them to the job queue.

#### Missed Tasks When System Was Down

The scheduler never auto-decides what to do with missed tasks. When the system comes back online it checks what was missed and sends you a Telegram message immediately:

```
System was down for 2h 14min (11:30am – 1:44pm).

Missed tasks:
1. Morning briefing (Personal Knowledge)
2. 3 scheduled social media posts (11:45am, 12:00pm, 1:00pm)
3. Chinese Product Finder daily scan (partial — 34% complete when stopped)
4. Language Learning daily lesson delivery

What should I do? Reply with numbers or say "suggest best options"
```

You decide what to do with each one. If you ask for suggestions, system replies with a recommendation per task with reasoning:

```
Suggestions:

1. Morning briefing — send now. Still useful even if late.
2. Social media posts — skip all 3. Wrong time slots, posting
   out of schedule looks unnatural.
3. Product Finder scan — resume from 34%. Partial data still valid,
   no need to restart from beginning.
4. Lesson delivery — send now. A late lesson is better than no lesson.

Reply with your decisions per item.
```

You reply with your decisions. System acts on each one. Nothing happens automatically — every missed task waits for your decision.

All scheduled tasks visible in one calendar view in the web app.

---

### 3 — Telegram Notification Service

Every application sends Telegram messages — alerts, briefings, progress updates, confirmations. All go through one central service.

**How Telegram works in this system:**
- A Telegram bot is created once via @BotFather — gives a bot token
- Bot token stored in settings, used by OpenClaw to send and receive messages
- You message the bot from your personal Telegram account
- Bot only receives messages sent directly to it — no access to your personal chats
- Delivery mode: webhook (Telegram calls your server instantly) — preferred over polling
- No session management, no QR scan, no reconnect complexity — Telegram bot API is stateless

**Message formatting:**
- Rich formatting using Telegram MarkdownV2 — bold, italic, code blocks, structured layouts
- Inline buttons for all approvals — tappable [Approve] [Modify] [Skip] — no typing needed
- Images sent as Telegram photos (post previews, generated images — up to 10MB)
- Videos never sent through Telegram — Drive link or web app link sent instead (Telegram bot limit is 50MB, most product videos exceed this)

**Reliability:**
- If delivery fails — messages queue and retry automatically, nothing lost
- Rate limiting handled centrally — one service manages Telegram API limits for all applications
- Consistent message style across all applications
- Incoming messages (your replies) routed to the correct application based on context

**Network and bandwidth:**
- Text messages: negligible bandwidth (a few KB each)
- Images: 100KB–5MB per image
- Videos: never through Telegram — sent as links only
- Your VPS internet handles all bot ↔ Telegram API traffic
- Your phone internet handles your personal Telegram app traffic
- Both are completely separate — one does not affect the other

---

### 4 — Claude API Gateway

Every application calls Claude. All calls go through one central gateway that enforces budgets, handles model tier selection, manages caching, and tracks all token usage. Full strategy detailed in the Claude API section above.

---

### 5 — Google Drive Service

Every application stores files on Google Drive. One central service handles all Drive operations — upload, download, folder organisation, authentication — across multiple accounts. No application talks to Drive directly — everything goes through this service.

---

**Folder Structure**

Every Drive account has the same folder structure. The system creates and manages it automatically — you never organise Drive manually.

```
AI Assistant/
  ├── Social Media Manager/
  │     └── [Business Name]/
  │           ├── Products/
  │           │     └── [Product Name]/
  │           │           ├── Images/
  │           │           ├── Videos/
  │           │           ├── Templates/
  │           │           └── Reviews/
  │           │                 ├── Images/    ← review screenshots
  │           │                 └── Videos/    ← video testimonials
  │           └── Categories/
  │                 └── [Category Name]/
  │                       ├── Images/
  │                       └── Videos/
  ├── AI Content Creator/
  │     └── [Channel Name]/
  │           ├── Scripts/
  │           ├── Visuals/
  │           └── Published/
  ├── Chinese Product Finder/
  │     └── Product Research/
  └── System Backups/
        └── Database/
```

**What lives in MongoDB instead of Drive (text/structured data):**
- Captions (all variants per product)
- AI prompts (image generation and caption generation)
- Product details (name, price, description, features, purchase link)
- Hashtag sets
- Posting rules and seasonal notes
- Posting frequency settings
- Review text quotes (the text content — screenshots/videos still go to Drive)

When a new account is added, the same structure is created on it automatically. Folders are only created when a file is first uploaded to that path — empty folders are not pre-created.

---

**File Index (internal database)**

The system maintains a file index in MongoDB. Every file across every Drive account has one record:
- File name and type
- Google Drive file ID
- Which Drive account it lives on
- Full folder path
- Which application, business, product, or category it belongs to
- Asset type (image / video / template / review-screenshot / review-video)
- File size and upload date
- Status (active / archived)

This index is what powers the unified view — the web app reads the index, not Drive directly, when browsing. Drive is only accessed when a file needs to be fetched for use or preview.

---

**Multiple Account Management**

- Add as many Drive accounts as needed — registered from web app via Google OAuth
- Each account given a label (e.g. "Main", "Overflow 1") for easy identification
- Each account shows: label, email, total space, used space, available space, date added
- Accounts can be Active or Paused — paused accounts receive no new uploads but existing files remain fully accessible

**Upload routing:**
- System always uploads to the account with the most available space
- Same folder path used on whichever account the file lands on
- A product's files may be spread across multiple accounts over time — this is fine because the file index always knows exactly where each file is
- From the web app the account boundary is invisible — all files appear in one unified view regardless of which account they physically live on

---

**Adding a New Account**

1. Settings → Google Drive Accounts → Add Account → OAuth login
2. System reads available space immediately
3. Same folder structure created on the new account as files are uploaded to it
4. Account joins the routing pool — new uploads start going there automatically
5. If files were queued (waiting because all accounts were full) — they upload to the new account immediately
6. Telegram confirmation: "New Drive account added — X GB available. Y pending files uploaded."

Old files stay exactly where they are. No migration required. The unified view continues working without any change.

---

**Storage Alerts**

| Threshold | Action |
|---|---|
| Any account at 80% | Telegram warning — time to add a new account |
| Any account at 95% | Urgent Telegram alert |
| Any account at 100% | Uploads to that account stop. Files that were routed there queue on server. System re-routes new uploads to other accounts. Alert sent with queued file count. Nothing lost. |

---

**Removing an Account**

An account with files on it cannot be removed directly.

1. Settings → Google Drive Accounts → select account → Remove
2. If files exist: system blocks removal, shows file count and total size
3. You must migrate files off it first — select a destination account, migration runs in background, file index updated
4. Once empty — removal allowed, OAuth token revoked, account removed from registry

---

**General Behaviours**
- Upload queue — if Drive is temporarily unavailable, files wait on the server and upload when connection restores. Nothing lost.
- Server is temporary workspace only — files deleted from server immediately after successful Drive upload confirmed
- Downloads for preview are cached briefly on server (30 minutes) then cleared — no permanent local copy

---

### 6 — Settings Store

All configuration lives in one place, organised into two levels. Every application reads from this store.

**Global settings** — apply across everything:
- API keys (Claude, Google, Meta, TikTok etc.)
- Alert thresholds (RAM, CPU, disk, Drive)
- Claude API monthly budget and model tier rules
- Telegram bot token and your personal chat ID

**App-specific settings** — only relevant to their own application:
- Social Media Manager: posting schedules, platform accounts, special days calendar
- AI Content Creator: channel configurations, style guides, posting schedules per channel
- Reminder & Calendar: Google Calendar account, morning briefing time
- Language Learning: active languages, lesson schedule, difficulty level per language
- Personal Knowledge: interest areas, RSS sources, briefing times
- Chinese Product Finder: category profiles, price bands, margin targets, MOQ limits

**How it works:**
- Web app settings page has two sections — Global and Applications
- Under Applications, each app has its own settings section
- An app can only read and write its own settings — never touches another app's settings
- Global settings readable by all apps, changeable only from the Global section
- Sensitive values (API keys) stored securely, never shown as plain text in the UI
- Change history kept — if something breaks after a settings change, you can see exactly what changed

---

### 7 — Image Generation Service

Shared service used by any application that needs AI-generated images. All requests go through this one service — no application calls Pippit.ai directly.

**Tool:** Pippit.ai via Puppeteer (browser automation)
- Pippit.ai has no public API — Puppeteer controls a Chromium browser to interact with it
- Chromium (snap) already installed on VPS
- Current model: Seedream 5 Lite (free tier)

**How it works:**
- Application adds an image generation job to the Job Queue
- Image Generation Service picks it up, opens Pippit.ai in Chromium via Puppeteer, submits the prompt and settings, waits for the result, downloads the image
- Image saved to server temporarily, then uploaded to Google Drive via Google Drive Service
- File index updated, application notified with the file reference
- All requests queue — only one generation runs at a time (Pippit.ai is one session, one browser)

**Why shared:**
- All applications that need image generation (Social Media Manager now, AI Content Creator later, others in future) use the same service
- If Pippit.ai changes their UI and automation breaks — one fix in one place, all applications benefit
- If a better tool with a real API is found later — swap the implementation in one service, nothing else changes

**Limitations:**
- One at a time — simultaneous requests queue, do not run in parallel
- Fragile to UI changes on Pippit.ai — needs maintenance if their interface updates
- Free tier may have daily generation limits — monitor usage

---

## Containers

### 1. OpenClaw (port 18888)
- AI brain of the system
- Connected to Telegram
- Talks to Claude API
- Triggers actions in Main App via internal Docker network
- Handles all Telegram command flows

### 2. Main App — Next.js (port 17777)
- Single unified web application
- No login (personal use only)
- Modules added over time as new pages/routes
- Internal API consumed by OpenClaw

---

## Data Storage — MongoDB

MongoDB already running on VPS (port 27017). All AI Assistant data goes here. One database, all applications share it with clearly named collections per application.

---

### Shared Collections (used across all applications)
- Job queue records
- Scheduler records and history
- Telegram message queue and conversation history
- System health logs and metrics
- Claude API usage records (per app, per day, per model tier)
- Google Drive account registry and file index
- Global and app-specific settings
- Audit log (what changed, when, triggered by what)

---

### Per-Application Collections

| Application | What Gets Stored |
|---|---|
| Social Media Manager | Post queue, platform accounts, posting history, competitor profiles, competitor activity history, ad intelligence records |
| AI Content Creator | Channels, idea pool, scripts, production pipeline state, style guides per channel, resource library, published video history |
| Reminder & Calendar | All reminders, recurring schedules, calendar events cache, reminder delivery history |
| Language Learning | Active languages, lessons library, notes, research queue, progress history per language |
| Personal Knowledge | RSS sources, briefing history, saved items, content ideas board |
| Chinese Product Finder | Full product structure (Niche → Family → Variant → Listing), keyword states, category profiles, supplier records, watchlist, daily list history |

---

### Data Retention Policy

**Knowledge data — keep indefinitely:**
These records have long-term business value. Deleting them loses real intelligence.
- Product records, supplier records, keyword history (Chinese Product Finder)
- Style guides and viral research (AI Content Creator)
- Personal notes (Language Learning, Personal Knowledge)
- Competitor profiles and their full history (Social Media Manager)
- Category profiles and version history (Chinese Product Finder)

**Operational data — time-limited retention:**

| Data | Retention |
|---|---|
| Daily product lists | 90 days |
| Daily briefing history | 90 days |
| Competitor activity feed | 90 days |
| Published post history | 12 months |
| Claude API daily usage records | 12 months |
| System health metrics | 90 days |
| Job queue completed records | 30 days |
| Telegram message history | 60 days |

**Archiving vs deleting:**
When operational records pass their retention period they move to an archive — still stored, not visible in normal views, not slowing down queries. Permanent deletion only after 2 years or manually from the web app. Nothing is hard-deleted automatically.

---

### Database Backup

Self-hosted MongoDB — backups handled entirely by the system. No external managed service.

**Where backups go:**
Google Drive — already in use, survives VPS failure, accessible from anywhere. Each backup is a compressed snapshot of the full database.

**Three-layer backup schedule:**

| Backup Type | Frequency | Copies Kept |
|---|---|---|
| Daily | Every day at 3am | Last 7 days |
| Weekly | Every Sunday | Last 4 weeks |
| Monthly | First day of each month | Last 6 months |

**Backup verification:**
After every backup completes — system confirms the file exists on Drive, is the expected size, and is not corrupted. Silent on success. Immediate Telegram alert on any failure.

**Alerts:**
- Backup failed → immediate Telegram alert
- Verification failed → immediate Telegram alert
- No backup in last 25 hours → Telegram alert (catches silent failures)
- Weekly confirmation every Sunday — "Backup healthy. Last 7 days all verified."

**Restore:**
Dedicated restore page in the web app. Shows all available backups with date, size, and verification status. Select a backup, confirm, system restores. Simple and accessible when actually needed.

**Storage estimate:**
Compressed MongoDB dump for a personal system starts at a few MB, grows slowly. Even after 2 years of heavy use — unlikely to exceed 2-3GB total for all backups combined.

---

## Telegram Bot Connection

Telegram bot API is stateless — no session, no QR scan, no reconnect complexity.

**Initial setup (one time only):**
1. Message @BotFather on Telegram
2. Create a new bot — get a bot token
3. Add token to settings and .env
4. Add your personal Telegram chat ID to settings (so bot knows who to message)
5. Done — bot is live immediately

**Delivery mode — webhook (preferred):**
Telegram calls your VPS webhook URL the instant a message is sent. Requires your VPS to have a public IP (it does). Instant delivery, no polling overhead.

**If webhook fails — polling fallback:**
OpenClaw polls Telegram API every few seconds for new messages. Slightly slower but works without any domain or SSL setup. Used as fallback if webhook has issues.

**If bot goes offline (container down):**
- Messages sent while container is down are queued by Telegram for up to 24 hours
- When container restarts — all queued messages delivered immediately
- No reconnect needed — bot token never expires
- Web app shows container status on system monitor page
- External uptime monitor (UptimeRobot) sends email alert if container is down

---

## Web App Structure

Single Next.js application (port 17777) housing all 6 applications plus shared system pages.

**Navigation: Sidebar**
Persistent left sidebar always visible. All 6 applications with their sub-pages always accessible. You always see where you are and can jump anywhere in one click.

**Home screen: Overview Dashboard**
Opening the app shows a live snapshot across everything — pending approvals, active alerts, today's stats. One glance shows what needs attention across all applications without visiting each one separately.

```
Sidebar (left, always visible)
│
├── Home — today's overview, pending approvals, active alerts
│
├── Social Media Manager
│     ├── Dashboard
│     ├── Post Queue
│     ├── Competitor Monitoring
│     ├── Image Generation
│     └── Analytics
│
├── AI Content Creator
│     ├── Dashboard
│     ├── Channels
│     ├── Idea Pool
│     ├── Production (active pipelines)
│     └── Published
│
├── Reminder & Calendar
│     ├── Reminders
│     └── Calendar
│
├── Language Learning
│     ├── Dashboard
│     ├── Lessons
│     └── Notes
│
├── Personal Knowledge
│     ├── Briefings
│     ├── Saved Items
│     └── Content Ideas
│
├── Chinese Product Finder
│     ├── Daily List
│     ├── Watchlist
│     ├── Categories
│     ├── Suppliers
│     └── Analytics
│
└── System
      ├── Monitor (health + Claude API usage)
      ├── Settings
      ├── Backup & Restore
      └── Telegram
```

---

## Application 1 — Social Media Manager

### Business Profiles

The entire Social Media Manager is organised around business profiles. Each business is a self-contained workspace with its own connected accounts, post queues, schedules, content categories, and competitors. Infrastructure underneath is shared — same Claude API, same Google Drive (separate folders per business), same database.

```
Business A (e.g. Electronics Store)
  ├── Facebook Pages: Page X, Page Y
  ├── Facebook Groups: Group 1, Group 2
  ├── Instagram: Account A
  ├── TikTok: Account A
  └── YouTube: Channel A

Business B (e.g. Clothing Brand)
  ├── Facebook Pages: Page Z
  ├── Instagram: Account B
  ├── TikTok: Account B
  └── YouTube: Channel B
```

Each business has completely independent post queues, schedules, content categories, competitor lists, special days priorities, and analytics.

**In the web app** — switch between businesses from the sidebar. Everything shown belongs to the currently selected business.

**On Telegram** — one business is set as active at a time. All Telegram interactions apply to the active business. Switch by saying "switch to clothing brand" or from the web app.

---

### Platform Account Management

| Platform | Accounts | Connected Through | Default on connect |
|---|---|---|---|
| Facebook Pages | Multiple — unlimited | Meta OAuth login | None activated — turn on manually |
| Facebook Groups | Multiple — unlimited | Same Meta login | None activated — turn on manually |
| Instagram | Multiple — unlimited | Via Facebook OR independent OAuth (both options) | None activated — turn on manually |
| TikTok | Single account | TikTok OAuth | Activated |
| YouTube | Single account | Google OAuth | Activated |

**Connecting accounts — web app only:**
Each platform has a Connect button. Click → redirected to platform login → authorise → redirected back. One-time setup per account.

One Meta login covers Facebook Pages, Facebook Groups, and any Instagram accounts linked to those pages.

**Instagram — two connection methods always available:**
- **Via Facebook** — system detects Instagram accounts linked to your Facebook Pages during Meta login. Activate the ones you want.
- **Independent** — add Instagram separately with its own OAuth. Works for accounts not connected to any of your Facebook accounts.

Both methods can be used at the same time. Each Instagram account shows which method it was connected through.

**Token expiry — handled proactively:**
- 7 days before expiry → Telegram alert to reconnect before interruption
- Expires before reconnect → affected scheduled posts paused (not cancelled), Telegram alert sent, posts resume once reconnected
- Each account shows token status in web app: Active / Expiring Soon / Expired

**Removing accounts:**
Disconnect any time from web app. Scheduled posts for that account move to pending reconnection state — nothing deleted.

---

### Feature 1 — Social Media Posting & Scheduling

### Platforms
- Facebook Pages (multiple) — post + manage
- Facebook Groups (multiple) — post only
- Instagram (multiple accounts) — video + photo
- TikTok — video + photo
- YouTube — upload with full metadata (title, description, thumbnail, tags)

### Media Storage

All media lives on Google Drive — managed by the central Google Drive Service (see Core Infrastructure). The Social Media Manager folder structure on Drive is:

```
AI Assistant/
  Social Media Manager/
    [Business Name]/
      Products/
        [Product Name]/
          Images / Videos / Templates / Reviews/Images / Reviews/Videos
      Categories/
        [Category Name]/
          Images / Videos
```

Captions, prompts, product details, hashtag sets, and posting rules are stored in MongoDB — not Drive. Drive holds files only (images, videos, template files, review screenshots/videos).

Files are routed automatically across multiple Drive accounts based on available space. The web app shows all files in one unified view — Drive account boundaries are invisible. Full multi-account management details are in Core Infrastructure — Google Drive Service.

---

### Content Organisation

All content for a business is organised into two completely separate tracks:

**Track 1 — Products** (selling oriented)
Showcasing specific products — features, prices, offers. Directly drives sales.

**Track 2 — Content Categories** (relationship oriented)
Non-product content that keeps the page feeling human and engaging. Visitors who only see product posts feel like they are on a shopping catalogue. Category posts create personality, trust, and audience connection.

The posting schedule mixes both tracks. The ratio is configurable per business — for example 40% product posts, 60% category posts. This mix creates the natural feel.

---

#### Product Library

Each product is a complete content package. Everything needed to post about it lives in one place.

```
Product — [Name]
  │
  ├── Details
  │     ├── Name, price, description
  │     ├── Key features and selling points
  │     ├── Purchase link / product page URL
  │     └── Tags and keywords
  │
  ├── Media
  │     ├── Images
  │     │     └── Variants — each variant has:
  │     │           ├── Label (e.g. "white background", "lifestyle shot", "price focus")
  │     │           ├── Type (different angle / crop / style / background / colour)
  │     │           ├── Drive file ID (pointer to actual file)
  │     │           ├── Notes (e.g. "works better during Ramadan")
  │     │           ├── Asset state (New / Used / Recently Used / Archived)
  │     │           ├── Usage count + last used date
  │     │           └── Performance data from posts using this variant
  │     │
  │     └── Videos
  │           └── Variants — each variant has:
  │                 ├── Label (e.g. "60 sec full demo", "15 sec hook", "price highlight")
  │                 ├── Type (different edit / length / format / opening / hook)
  │                 ├── Drive file ID (pointer to actual file)
  │                 ├── Notes (e.g. "best for TikTok and Reels")
  │                 ├── Asset state (New / Used / Recently Used / Archived)
  │                 ├── Usage count + last used date
  │                 └── Performance data from posts using this variant
  │
  ├── Captions
  │     ├── Multiple saved captions per context (launch, promo, feature highlight)
  │     └── Platform-specific versions (Facebook, Instagram, TikTok, YouTube)
  │
  ├── Image Templates
  │     └── Banner and ad templates specific to this product
  │
  ├── AI Prompts
  │     ├── Image generation prompts for this product
  │     └── Caption generation prompts (generate new captions using product details)
  │
  ├── Hashtag Sets
  │     └── Multiple sets for different contexts (launch, promo, general)
  │
  ├── Seasonal Notes
  │     └── e.g. "Best in winter — push harder November onwards"
  │
  ├── Posting Rules
  │     └── e.g. "Never post on Fridays" / "Only post during Ramadan"
  │
  ├── Posting Frequency
  │     ├── How often: X times per week (default: 1x)
  │     ├── Until: optional end date — reverts to default after this date
  │     └── Note: optional reminder to yourself (e.g. "ads running this month")
  │
  ├── Client Reviews
  │     ├── Text reviews (quote + reviewer name/alias + source)
  │     ├── Screenshots (Telegram chats, Facebook comments, Messenger, email)
  │     ├── Video testimonials
  │     ├── Rating (optional — e.g. ⭐⭐⭐⭐⭐)
  │     ├── Status per review: Pending Approval / Approved / Rejected / Used / Archived
  │     └── Privacy flag: show reviewer name / hide name / use alias
  │
  ├── Status
  │     └── Active / Paused / Discontinued
  │
  ├── Milestone Rules
  │     └── Product-level rules (override business defaults — see Milestone Actions section)
  │
  └── Post History
        └── All posts made about this product — each post record stores:
              ├── Platform and account posted to
              ├── Posted date and time
              ├── Asset used (Drive file ID + variant ID)
              ├── Caption used
              ├── Platform-specific details (YouTube title, tags, thumbnail etc)
              ├── Performance data — full sync history with timestamps (likes, views, comments, shares tracked at every sync point — not just current totals)
              ├── Milestone rules active at time of posting
              ├── Milestone actions triggered (which milestone hit, when, what action was taken)
              └── Post status (Draft / Scheduled / Published / Failed / Archived)
```

**Posting frequency from Telegram — natural language, no format to remember:**
```
"post Samsung TV more often"
"I'm running ads on the TV, push it more this month"
"post this product 3 times a week until end of November"
"reduce posting on the speaker, it's out of stock"
```
System understands intent from whatever you say. Asks one short question only if something is unclear.
Frequency end date reminder sent via Telegram 2 days before it expires.

---

#### Content Category Library

Non-product content for audience engagement. Each category has its own asset library. No product details, no pricing, no selling.

**Built-in categories:**
- Special Days (Eid greetings, Independence Day, Mother's Day, etc.)
- Fun / Entertainment (relatable content, humour, trending topics)
- Educational (tips, how-to, industry knowledge)
- Notices (important announcements, business updates)
- Behind the Scenes (team, process, culture)
- Motivational / Inspirational

Custom categories can be added per business.

**Each category contains:**
```
Category — [Name]
  │
  ├── Media (images, videos relevant to this category)
  ├── Captions (pre-written captions for this content type)
  ├── Image Templates
  ├── AI Prompts (for generating content in this category's style)
  ├── Hashtag Sets
  ├── Seasonal Notes (e.g. "Special Days — activate around relevant dates")
  ├── Posting Rules (e.g. "Fun posts — only on weekends")
  ├── Status (Active / Paused)
  └── Post History
```

**Special Days** category has an additional built-in calendar — Bangladesh national days, Islamic calendar (Eid, Ramadan, etc.), and international days. Configurable advance reminder per day so you prepare content in time.

### Post Scheduling System

Every platform entity — each Facebook Page, each Facebook Group, each Instagram account, TikTok, YouTube — has its own independent schedule. Each entity is managed separately with its own slots, its own queue, and its own recycling settings. This allows completely different posting strategies per entity.

```
Business A
  ├── Facebook Page X → own slots (8am product video / 12pm fun post / 8pm product image)
  ├── Facebook Page Y → own slots (9am product image / 7pm product video)
  ├── Facebook Group 1 → own slots (10am notice / Friday announcement)
  ├── Instagram Account A → own slots (10am product image / 6pm category post)
  ├── TikTok → own slots (11am product video / 9pm fun post)
  └── YouTube → own slots (Sunday 2pm weekly upload)
```

**Cross-posting — Linked Slots:**
When you want the same post on multiple entities simultaneously — create a linked slot. Define once, select which entities it applies to. All linked entities post the same content at the same time.

```
Linked Slot — "Daily 8pm product video"
  Applied to: Facebook Page X, Facebook Page Y, Instagram A
  → Same post goes to all three simultaneously
```

Linked slots are optional. Each entity can have fully independent slots alongside any linked ones.

**Schedule view in web app:**
Each entity shown as its own column. Manage slots per entity independently. Linked slots shown with a visual indicator across the columns they apply to.

---

The schedule is a set of defined slots. Each slot is a rule that repeats on a pattern.

#### Slot Structure

Every slot is fully configurable. Each slot has a set of rules evaluated top to bottom — first matching rule wins. This makes the slot condition-based and dynamic.

```
Slot
  ├── Day pattern
  │     ├── Every day
  │     ├── Weekdays only (Mon–Fri)
  │     ├── Weekends only (Sat–Sun)
  │     ├── Specific days (e.g. Mon, Wed, Fri)
  │     └── Monthly (e.g. 1st of every month)
  │
  ├── Time (e.g. 08:00 PM)
  │
  ├── Per-platform content (each platform can have different content in the same slot)
  │     ├── Facebook → Product video (16:9)
  │     ├── Instagram → Product image (1:1)
  │     └── TikTok → Short product clip (9:16)
  │
  └── Rules (evaluated top to bottom — first match wins)
        │
        ├── Rule 1
        │     ├── Condition: IF today is a special day
        │     ├── Post: Greetings image
        │     ├── From: Special Days category
        │     └── Asset selection: newest / random / specific
        │
        ├── Rule 2
        │     ├── Condition: IF a product has posting frequency > 2x this week
        │     │              AND has not posted today
        │     ├── Post: Product video
        │     ├── From: that product
        │     └── Asset selection: queue order
        │
        ├── Rule 2b (example — dedicated review slot)
        │     ├── Condition: IF it is Wednesday
        │     ├── Post: Client review
        │     ├── From: Reviews — [specific product / all active products]
        │     ├── Media type: Images only / Videos only / Both
        │     └── Asset selection: random / newest / queue order
        │
        ├── Rule 3
        │     ├── Condition: IF it is Friday
        │     ├── Post: Fun post
        │     ├── From: Fun / Entertainment category
        │     └── Asset selection: random
        │
        └── Default Rule (runs when no condition above matched)
              ├── Post: Product video
              ├── From: All active products (rotate)
              └── Asset selection: queue order
```

#### Available Conditions

```
Date and time based:
  - Today is a special day (specific event or any event)
  - Day of week is [specific day]
  - Within X days of a special event
  - Current season (Ramadan, winter, summer, etc.)

Product based:
  - Product has posting frequency set above X per week
  - Product has not been posted in last X days
  - Product has new assets not yet posted
  - Product status is Active
  - Product has approved reviews not yet posted
  - Product has approved review images not yet posted
  - Product has approved review videos not yet posted

Content availability:
  - Queue for this source has content available
  - Queue is running low (below X items)
  - No new content available — fall back to next rule

Content source options (used in the "From" field of any rule):
  Products:
    - A specific product
    - All active products (rotate)
    - Products with high posting frequency
  Product Reviews:
    - Reviews for a specific product
    - Reviews across all active products (rotate)
    - Media type filter: Images only / Videos only / Both
  Categories:
    - A specific category (Fun, Educational, Notice, etc.)
    - Special Days category

Manual override:
  - One-time condition set for a specific date
```

#### Special Event Conflict Handling

When a special event activates at the same time as a regular slot — if a rule covers it, it is handled automatically. If no rule covers it, system sends a Telegram notification:

```
"Eid ul-Fitr is tomorrow. Your special event (8pm greetings) conflicts
with regular 8pm product video slot.

1. Replace with greetings for this day only
2. Post both — greetings at 8pm, product video at 8:30pm
3. Skip special event this time
4. Skip regular slot this time"
```

Regular slot continues as normal the next day — only that one occurrence is affected.

#### Content Recycling — Assets Not Posts

When a slot's queue runs low or empty — the system reuses assets (images, videos) with freshly generated captions. Never the exact same post twice. Same media, new caption each time.

#### Managing Slots

**Web app** — visual schedule grid showing all slots across the week. Full slot editor with condition builder. Rules listed in order — drag to reorder priority. Preview shows what the slot would post today given current conditions.

**Telegram — natural language:**
```
"add a slot every day at 8am for product videos"
"add a rule to the 8pm slot — if it's Friday post fun content"
"what will post tonight at 8pm?"
"change the default for the morning slot to product images"
"skip tomorrow's 10am post"
```

---

#### Content States

Every asset and post has a state that drives scheduling logic, recycling, and what appears in the web app.

**Asset states:**
```
New           — uploaded, never used in any post
Used          — used in at least one post before
Recently Used — used within last X days (default: 14 days) — skipped in auto-scheduling
Archived      — kept for records, excluded from automatic scheduling
```

Scheduler always prefers New assets first. Falls back to Used assets — skipping Recently Used to avoid repetition.

**Post states:**
```
Draft         — created but not finalised or scheduled
Scheduled     — assigned to a slot or time, waiting to go out
Recyclable    — recycled post scheduled, can be displaced by new content
Publishing    — currently being sent to platform(s)
Posted        — successfully published, performance tracking active
Failed        — attempted but failed after all retries
Archived      — old posts kept for records, removed from active views
```

---

#### Recycling Control

Recycling is Off by default everywhere. Must be explicitly turned on where it makes sense. Not all products or categories need recycling — and even where it is enabled, it can be paused at any time.

**Controlled at four levels (each can override the one below):**

```
Business level (master switch)
  └── Recycling: On / Off — overrides everything below

Slot level
  └── Allow recycling: Yes / No — overrides product/category setting for this slot

Product level
  └── Recycling: On / Off (default: Off)

Category level
  └── Recycling: On / Off (default: Off)
```

Examples of sensible defaults:
- Special Days category — Off (never repost last year's Eid greeting)
- Fun Posts category — On (evergreen content, recycling is fine)
- New product launch — Off (only fresh content during launch period)
- Evergreen products — On

The 12-hour recycling preparation flow only triggers when recycling is enabled at both the product/category level AND the slot allows it. If recycling is off — slot is skipped and you get an alert that no content is available.

**From Telegram:**
```
"turn on recycling for Samsung TV"
"stop recycling fun posts"
"pause all recycling this week"
"is recycling on for the 8pm slot?"
```

---

#### Milestone-Based Actions

When a published post hits a performance milestone — the system detects it, prepares the defined action, and asks for your approval. Nothing happens automatically without your confirmation.

**How milestones are checked:**
Performance data (likes, views, comments, shares) is synced periodically from each platform's API. After every sync, the system checks all recent posts against their applicable milestone rules.

---

#### Performance Data Tracking

**What each platform provides:**

| Platform | Available Metrics | Via |
|---|---|---|
| Facebook Pages | Likes, comments, shares, reach, impressions | Meta Graph API |
| Facebook Groups | Post status only (no engagement metrics) | Not available via API |
| Instagram | Likes, comments, saves, reach, impressions, video plays | Meta Graph API |
| TikTok | Views, likes, comments, shares, play time | TikTok API |
| YouTube | Views, likes, comments, watch time, impressions | YouTube Analytics API |

Facebook Groups have no metrics — Meta does not expose group post engagement via API. Group posts are tracked for status (posted / failed) only.

**Sync frequency:**

| Post age | Sync frequency |
|---|---|
| Last 7 days | Every 6 hours |
| 7–30 days old | Once daily |
| Older than 30 days | Once weekly |

Recent posts sync frequently because engagement moves fast in the first week. Older posts settle and need less frequent updates.

**What gets stored:**
Every metric is stored with a timestamp on every sync — not just the current count but the full history of how it grew. This lets you see when a post peaked, how fast engagement grew, and whether a repost outperformed the original.

```
Post performance history:
  ├── [2026-04-05 08:00] likes: 4, views: 120, comments: 1
  ├── [2026-04-05 14:00] likes: 11, views: 340, comments: 3
  ├── [2026-04-05 20:00] likes: 23, views: 580, comments: 5  ← milestone hit (20 likes)
  └── [2026-04-06 08:00] likes: 28, views: 710, comments: 6
```

After every sync — milestone rules checked against the latest numbers. If a milestone is newly crossed — action prepared and approval sent.

---

**Rule Hierarchy — Three Levels:**

```
Business level (defaults for all products)
  └── e.g. "Any post with 50+ likes → increase posting frequency"

Product level (overrides business defaults for this product only)
  └── e.g. "Samsung TV: any post with 20+ likes → repost with different variant"

Post level (one-time override for a specific post only)
  └── Set at post creation time for special cases
```

If a product has no rules defined → business-level rules apply.
If a product has its own rules → product rules apply instead (full override, not merge).
Post-level rules apply only to that one post regardless of product or business rules.

---

**Available Milestone Triggers:**

| Trigger | Example |
|---|---|
| Likes reach X | 20 likes, 50 likes, 100 likes |
| Views reach X | 500 views, 1000 views |
| Comments reach X | 10 comments |
| Shares reach X | 5 shares |
| Engagement rate reaches X% | 5% engagement rate |
| Any combination | 20 likes AND 5 shares |

---

**Available Actions:**

**1. Repost with variant**
Repost the same product using a different asset variant. You explicitly define what changes.

```
Action: Repost with variant
  ├── Asset: different variant / same variant / let system pick least recently used variant
  ├── Caption: same caption / new caption / modified caption (you specify changes)
  ├── Platform-specific changes (per platform, optional):
  │     ├── Facebook: same / different caption
  │     ├── Instagram: same / different caption / different hashtags
  │     ├── TikTok: same / different caption
  │     └── YouTube: same title / modified title / same thumbnail / new thumbnail / same tags / updated tags
  ├── Post to: same platforms / specific platforms only
  └── Timing: immediately / after X days / add to queue
```

**2. Increase posting frequency**
Temporarily increase how often this product appears in the schedule.

```
Action: Increase posting frequency
  ├── New frequency: X times per week
  ├── Duration: X days / until further notice
  └── Note added to product: (e.g. "auto-increased — post hit 50 likes on [date]")
```

---

**Approval flow:**

When a milestone is hit and action is prepared:

```
Telegram: "Samsung TV post hit 20 likes ✓
           [post preview]

           Action ready: Repost with '15 sec hook' variant
           Caption: same as original
           Platforms: Facebook Page X, Instagram A
           Timing: tomorrow 8pm slot

           Approve / Modify / Skip"
```

You reply:
- **Approve** → action queued, goes through normal post approval flow before publishing
- **Modify** → system asks what to change (different variant? different time? different platform?)
- **Skip** → action dismissed, milestone marked as seen, no further alerts for this post

One milestone can only trigger one action — it does not fire again for the same post at the same milestone level.

---

**Setting up milestone rules:**

Web app — Rules section per product and per business settings:
- Add rule: select trigger (likes / views / etc + threshold) → select action → configure action details → save
- Rules listed in order, active/inactive toggle per rule
- Preview: "If this rule fires — here is what the prepared action will look like"

Telegram — natural language:
```
"if any Samsung TV post gets 20 likes, repost it with a different video"
"set a rule — if any post gets 50 likes, increase posting frequency for that product"
"what milestone rules do I have for Samsung TV?"
"remove the 20 likes rule from Samsung TV"
```

---

#### Content Lifecycle — Queue Management Flow

**Stage 1 — Early warning (queue running low)**
When a product or category queue drops below threshold (default: 3 posts remaining):
```
Telegram: "Samsung TV queue is running low — 2 posts remaining.
Add new content to avoid recycling."
```

**Stage 2 — Queue empty, slot approaching (T-12 hours)**
12 hours before a slot with no new content available:
```
Telegram: "No new content for Samsung TV — 8pm slot tonight.
Should I prepare a recycled post as backup?"
```

**If you say yes:**
System prepares recycled post — picks best asset not used recently, generates fresh caption using product details. Sends full preview for approval:
```
Telegram: "Recycled post ready for review.

Product: Samsung TV 43"
Asset: [product photo from March]
Caption: [newly generated caption]
Platforms: Facebook Page X, Instagram
Slot: Tonight 8pm

Approve / Change caption / Pick different asset / Skip this slot"
```
You approve or request changes. System adjusts and resubmits. Once approved — post scheduled with state: Recyclable.

**If you say no:**
Slot marked empty. System skips it. You handle manually.

**Stage 3 — New content added after recycled post is scheduled**
If new content is added after a Recyclable post is scheduled but before the slot fires:

- **Slot > 2 hours away** — auto-swap. New content takes the slot. Recycled post moves to next available slot. Telegram notification:
```
"New asset added for Samsung TV.
Moved recycled post to next slot.
New content scheduled for tonight 8pm ✓"
```

- **Slot < 2 hours away** — ask first:
```
"New asset added but tonight's 8pm slot is in 90 minutes.
Swap to new content tonight or use it in next slot?"
```

New content always gets priority. Recycled post never cancelled — moves to next available slot.

**Failed post handling:**
```
Post fails → retry automatically up to 3 times (15 min gaps)
All retries fail → status: Failed → Telegram alert
→ In web app: options to retry manually / reschedule / skip
→ Slot continues normally for next post — nothing blocked
```

---

### Post Creation Flow

For manually creating a specific post outside the automated slot system — a one-off post, announcement, or something for a specific time.

#### From the web app

```
Step 1 — Select content source
  ├── From a product (select product → browse its assets)
  ├── From a category (select category → browse its assets)
  └── Create new right here (upload / generate / URL import)

Step 2 — Select asset(s)
  ├── Single image
  ├── Multiple images (carousel)
  ├── Video
  └── Text only (no media)

Step 3 — Select platforms and accounts
  Tick which platforms and specific pages/accounts this post goes to.
  Each ticked platform shows its own settings panel.

Step 4 — Write content
  ├── Mode 1 — Write once, auto-adapt per platform
  │     Write one caption → system generates per-platform versions
  │     Each shown side by side — edit any freely
  │     Or select from saved captions in the product/category library
  └── Mode 2 — Write separately per platform
        Each platform has its own independent content editor

Step 5 — Platform-specific settings
  ├── Facebook Pages: select which pages, post type (image/video/carousel/text)
  ├── Facebook Groups: select which groups
  ├── Instagram: account selection, format compatibility check
  ├── TikTok: privacy (public/friends/private), allow comments/duet/stitch
  └── YouTube: title, description, tags, thumbnail, category, visibility
               (public / unlisted / private / scheduled)

Step 6 — Video handling (only if video selected)
  ├── System detects: aspect ratio, length, file size, resolution
  ├── Flags incompatibilities per platform:
  │     "This video is 16:9. Instagram Reels requires 9:16. Auto-crop or skip Instagram?"
  ├── Auto-crop option — you approve the cropped version before it is used
  ├── Cover frame selection (TikTok, Instagram Reels)
  └── YouTube thumbnail:
        ├── Pick a frame from the video
        ├── Upload custom image
        └── Generate using Image Generation

Step 7 — Apply branding (optional)
  Skip / Apply logo / Apply watermark / Select preset

Step 8 — Schedule
  ├── Post now (instant)
  ├── Specific date and time
  ├── Add to queue — next available matching slot
  └── Save as draft — finish later

Step 9 — Review and confirm
  Preview per platform. Confirm → saved to schedule, queue, or draft.
```

#### Instant Posting

Publishes immediately. Available from both web app and Telegram.

One confirmation always shown before publishing — never goes out without your explicit "yes" at that exact moment.

**From Telegram:**
```
You: "post this now to Electronics page"
[attach image]
System: "Caption?"
You: "New arrivals just landed"
System: "Posting now to Facebook Page X — confirm?"
You: "yes"
System: "Posted ✓ — Facebook Page X"
```

**Multi-platform instant post:**
All platforms publish simultaneously. If one fails while others succeed:
```
System: "Posted ✓ — Facebook Page X, Instagram A
         Failed — TikTok (connection error)
         Retry TikTok now or schedule for later?"
```
Successful platforms are not affected by a failure on another.

**YouTube** — never instant. Always requires full scheduling flow (title, tags, thumbnail, visibility are required).

After instant posting — post record created immediately, performance tracking starts.

---

### Platform-Specific Post Requirements

Each platform has different post types, required fields, media specs, and unique features. The system enforces these per platform and flags incompatibilities before scheduling.

---

#### Facebook Pages

**Post types:**
| Type | Media | Notes |
|---|---|---|
| Photo post | 1–10 images | Single or album |
| Video post | 1 video | Up to 240 min |
| Reel | 1 vertical video | 9:16, up to 90 sec |
| Story | Image or video | Disappears after 24h, up to 20 sec video |
| Carousel | 2–10 images/videos | Each card has own link |
| Link post | URL | Auto-generates preview card |
| Text only | None | No media required |

**Required fields:**
- Caption (optional but always recommended)
- Post type selection

**Optional fields:**
- First comment — post the hashtags in the first comment instead of the caption (keeps caption clean)
- Location tag
- Feeling / activity tag
- Alt text per image

**Character limits:**
- Caption: 63,206 characters
- Practical recommendation: keep under 300 for feed, under 125 for best reach

**Scheduling:**
- Native scheduling via Meta API (post goes out even if server is down at that moment)
- Reschedule or cancel anytime before publish time

**What the system handles:**
- First comment auto-posted immediately after publish (for hashtag sets)
- Album ordering — drag to reorder before scheduling
- Carousel link per card — configurable

---

#### Facebook Groups

**Same post types as Facebook Pages except:**
- No Reels (Groups do not support Reels)
- No Stories
- No Carousel via API (photo album only)
- No native scheduling via API — system queues and posts at exact time using its own scheduler

**Required fields:**
- Caption

**Limitations:**
- Group posting via API only works if the app has group posting permission
- Private groups require admin approval of the app
- No analytics returned from groups (Meta does not expose group post metrics via API)

**What the system handles:**
- Posts timed by system scheduler
- Failed posts flagged immediately — Telegram alert sent

---

#### Instagram

**Post types:**
| Type | Media | Specs |
|---|---|---|
| Feed photo | 1 image | Square 1:1, portrait 4:5, landscape 1.91:1 |
| Feed carousel | 2–10 images/videos | All must share same aspect ratio |
| Feed video | 1 video | Up to 60 min, 4:5 portrait recommended |
| Reel | 1 vertical video | 9:16, 15 sec – 15 min |
| Story | Image or video | 9:16, image shown 5 sec, video up to 60 sec |

**Required fields:**
- Caption (optional for Stories)
- Account selection
- Format selection (Feed / Reel / Story)

**Optional fields:**
- Hashtags (up to 30 — system warns if over limit)
- First comment for hashtags (same as Facebook — keeps caption clean)
- Alt text per image
- Location tag
- Collaboration tag (collab posts)
- Cover frame for Reels

**Character limits:**
- Caption: 2,200 characters
- Practical recommendation: under 150 for feed posts

**Media checks the system runs automatically:**
- Aspect ratio per format (flags mismatch, offers auto-crop)
- Video length per format (flags if too long)
- File size limit (8MB images, 4GB video)
- Carousel ratio consistency (all cards must match)

**What the system handles:**
- Auto-crop offered when ratio is wrong — preview shown before accepting
- Cover frame selection for Reels
- Story scheduling (disappears after 24h — system notes expiry)

---

#### TikTok

**Post types:**
| Type | Media | Specs |
|---|---|---|
| Video | 1 vertical video | 9:16, 3 sec – 10 min (up to 60 min for some accounts) |
| Photo slideshow | 2–35 images | Shown as swipeable slides with music |

**Required fields:**
- Caption
- Privacy setting

**Optional fields:**
- Cover frame (thumbnail shown before play)
- Allow comments (on/off)
- Allow duet (on/off)
- Allow stitch (on/off)
- Hashtags (no hard limit but 5–8 recommended)
- Mention
- Location

**Privacy settings:**
- Public — visible to everyone
- Friends — visible to mutual followers only
- Private — only you

**Character limits:**
- Caption: 2,200 characters including hashtags

**Media checks the system runs automatically:**
- Aspect ratio (9:16 required — flags and offers auto-crop)
- Video length (flags if under 3 sec or over 10 min)
- File size (under 4GB)

**What the system handles:**
- Cover frame selection — pick a frame from the video
- Privacy and interaction settings saved as default per TikTok account, overridable per post
- Photo slideshow ordering — drag to reorder before scheduling

---

#### YouTube

**Post types:**
| Type | Media | Notes |
|---|---|---|
| Standard video | 1 video | Any length, 16:9 recommended |
| Short | 1 vertical video | 9:16, up to 60 sec — auto-classified as Short |

**Required fields:**
- Title (up to 100 characters)
- Description
- Category
- Visibility setting
- Thumbnail

**Optional fields:**
- Tags (up to 500 characters total)
- Playlist — assign to one or more playlists
- Language
- Recording date and location
- Chapter markers — timestamps added to description (format: 0:00 Intro, 1:23 Section Name)
- End screen — added last 20 seconds (subscribe, video link, playlist link)
- Cards — clickable overlays at any point in the video
- Made for kids toggle
- Age restriction
- Allow comments (on/off/hold for review)
- Allow embedding

**Visibility settings:**
- Public — immediately visible
- Private — only you
- Unlisted — anyone with the link
- Scheduled — public at a set date and time

**Thumbnail:**
- Required — never post without one
- Options: pick frame from video / upload custom image / generate via Image Generation
- Recommended size: 1280×720 (16:9)

**Character limits:**
- Title: 100 characters
- Description: 5,000 characters
- Tags: 500 characters total

**YouTube-specific rule:**
YouTube never posts instantly. Every YouTube post goes through the full scheduling flow — title, description, thumbnail, visibility, and category are all required before the post can be queued. This is enforced in both web app and Telegram flows (Telegram redirects to web app for YouTube).

**What the system handles:**
- Shorts auto-detection — if video is 9:16 and under 60 sec, system flags it as a Short candidate
- Chapter markers — user writes timestamps in natural format, system formats correctly for YouTube description
- Thumbnail generation via Image Generation is available inline during post creation
- Playlist management — create, rename, assign from web app

---

#### From Telegram — conversational flow

For quick posts. System asks only what is missing.

```
You: "schedule this video"
System: "Which business?"
You: "electronics"
System: "Which platforms?"
You: "facebook and tiktok"
System: "Which Facebook pages — Page X, Page Y, or both?"
You: "both"
System: "Time?"
You: "tomorrow 6pm"
System: "Caption?"
You: "same as last week's product video"
System: "Used last Friday's caption. Scheduled ✓"
```

YouTube and complex posts redirect to web app:
```
System: "YouTube needs title, tags, and thumbnail.
         Opening in app — [link to pre-filled form]"
```

#### Post States After Creation
- **Scheduled** — specific date and time set
- **Queued** — added to next available slot
- **Draft** — saved, not yet scheduled
- **Posted** — published, performance tracking active

---

### Platform Flexibility Per Business

Not every business uses every platform. When setting up a business only the relevant platforms are activated. A business with no YouTube simply does not have YouTube activated — it never appears in that business's slots, post creation, or settings. Only active platforms are visible and usable for each business.

---

### Post Approval Flow

Applies to AI-prepared content — recycled posts with new captions, AI-generated captions for scheduled posts. Manually created posts need no approval since you created them directly.

**From Telegram — quick approval:**
```
System: "Recycled post ready for Samsung TV — 8pm tonight.

Caption: 'Still one of our best sellers — Samsung 43" TV.
          Full HD, smart features, great price. DM to order.'
Platforms: Facebook Page X, Instagram A

Approve / Change caption / Different asset / Skip"

You: "approve"
System: "Scheduled ✓"

You: "change caption"
System: "What should change?"
You: "make it shorter and add a question at the end"
System: [new caption shown] "Better? Approve / Change again"
```

**From web app — full edit:**
Full post editor opens with AI-prepared content. Edit anything freely — caption, asset, platforms, time. Save when satisfied.

**Approval never expires silently:**
If you do not respond — post stays pending, slot is skipped. System never posts without your approval. One follow-up reminder sent after 2 hours of no response.

**Approval queue:**
All posts waiting for approval shown in a dedicated section in the web app. Review and approve multiple at once.

---

### Telegram Interaction — Social Media Manager

All interactions via Telegram use natural language. No command syntax required.

Examples:
- "What's scheduled for today?"
- "Skip the next post"
- "Add this image to health products"
- "What special days are coming this week?"
- "How is my Facebook page doing?"
- "What should I post this week?"

AI understands intent from natural conversation. Commands are optional shortcuts — natural language always works.

---

### Asset Management

All assets for all products and categories are managed here. Available from two entry points — the full asset library view and directly from within a slot. Both write to the same storage. Adding an asset from anywhere makes it immediately available everywhere.

#### Asset Sources

| Source | Web App | Telegram |
|---|---|---|
| Manual upload | ✓ — file browser, bulk upload | ✗ |
| Telegram send | ✗ | ✓ — send media + say where it belongs (files up to 50MB — larger files use web app or Drive import) |
| Google Drive import | ✓ — browse folder structure, select files | ✗ |
| AI image generation | ✓ — prompt editor, preview, approve | ✓ — conversational flow |
| AI caption generation | ✓ — multiple variations, keep what you like | ✓ — sent back for review |
| URL import | ✓ — paste URL, select destination | ✓ — send URL + say where it belongs |
| Client review entry | ✓ — review form, screenshot upload | ✓ — forward screenshot or type review |

#### AI Image Generation (via Pippit.ai)

Image generation is one asset source — accessible from the asset library or directly from a slot. No separate section.

- Pippit.ai has no public API — uses Puppeteer browser automation
- Chromium (snap) already installed on VPS
- Current free model: Seedream 5 Lite via Pippit.ai

**Image types — purpose based, not style based:**
Each type defines what the image is for. Style is always a separate input — a festival image can be cartoon, text-based, or realistic. The type does not dictate the style.

| Image Type | Purpose |
|---|---|
| Product image | Showing the product |
| Lifestyle image | Product in use or real-world context |
| Review / testimonial | Client review turned into a shareable visual |
| Festival / greeting | Special days and events |
| Fun / entertainment | General engagement content |
| Notice / announcement | Business updates, offers, alerts |
| Educational | Tips, how-to, informational content |

Custom types can be added from settings.

**Fields per image type:**

Every type always has:
- **Prompt** (required — always, every type)
- **Style** (required — realistic / cartoon / illustration / text-based / mixed / custom)

Additional fields per type:
```
Product image:
  Required: Product photos (1 or more)
  Optional: Model reference, pose reference, background reference, style notes

Lifestyle image:
  Required: Product photos
  Optional: Setting description, model reference, mood/lighting reference

Festival / greeting:
  Required: Festival or event name
  Optional: Colour scheme, reference image

Review / testimonial:
  Required: Review text (the quote), product linked to
  Optional: Reviewer name or alias, star rating, source (Facebook / Telegram / etc.), background/colour scheme

Fun / entertainment, Notice, Educational:
  Optional: Reference image, colour scheme, additional style notes
```

**Generation flow — one image at a time:**

System generates one image, you review, then decide if you want another.

**Web app:**
Select image type → system shows required and optional fields for that type → fill in → generate → branding optional → preview → approve / modify / reject → if approved saved to library → "Generate another?"

**Telegram:**
```
You: "generate a product image for Samsung TV"
System: "Prompt?"
You: "TV on a modern living room shelf, evening lighting"
System: "Style? (realistic / cartoon / illustration / text-based / mixed)"
You: "realistic"
System: "Product photos? (send 1 or more)"
You: [sends 2 photos]
System: "Model reference? (send or skip)"
You: "skip"
System: "Generating..."
→ One image sent to Telegram
System: "Approve / Modify / Reject / Generate another"
You: "modify — warmer lighting"
System: "Generating updated version..."
→ Updated image sent
System: "Approve / Modify / Reject / Generate another"
You: "approve"
System: "Saved to Samsung TV library ✓  Generate another?"
```

Temp versions kept during session. Auto-cleared after 1 hour of inactivity.

**Proactive generation when queue is low:**
When queue drops below threshold — system notifies and asks what to do first. Only generates if you choose to.

```
Telegram: "Samsung TV image queue is low — 2 posts remaining.

1. Generate new images (AI)
2. Recycle existing content
3. I'll add content myself
4. Skip slots until I add content"
```

Option 1 only shown if AI generation is enabled for this product (configurable per product/category, default Off). If you choose generate — starts the one-at-a-time generation flow above. Never generates automatically without your instruction.

**AI Caption Generation:**
Claude generates captions from product details — key features, selling points, price, audience. Multiple variations at once. Review in web app or Telegram, keep what you like, discard the rest. Saved to product caption library.

---

#### Special Days & Festival Flow

Every festival and special day in the calendar has its own profile. System reminds you at least 2 days before (configurable per festival) and starts a conversation to plan the post. Image is always freshly generated — the profile saves the setup, never the output.

**Festival profile — what gets saved:**
```
Festival Profile — Eid ul-Fitr
  ├── Advance reminder: 2 days (configurable)
  ├── Setup (last used):
  │     ├── Post type (greeting / promotional / fun)
  │     ├── Message and concept
  │     ├── Style (realistic / cartoon / text-based / etc.)
  │     ├── Reference images
  │     ├── Colour scheme
  │     ├── Platforms and pages
  │     └── Schedule time
  └── Post history — all past festival posts with performance data
```

Profile stores the recipe — never the final image. Every year = fresh generation.

---

**Scenario 1 — First time this festival is handled:**

```
Telegram: "Eid ul-Fitr is in 2 days. Do you want to post anything?"

You: "yes"
System: "What kind of post? (greeting / promotional / fun / educational)"
You: "greeting"
System: "Which platforms and pages?"
You: "all Facebook pages and Instagram"
System: "Main message or feeling to convey?"
You: "warm Eid wishes, simple and elegant"
System: "Style? (realistic / cartoon / illustration / text-based)"
You: "text-based with a nice background"
System: "Reference image? (send or skip)"
You: [sends reference]
System: "Colour scheme?"
You: "gold and green"
System: "Got it. Generating fresh image now."
→ One image generated → sent for review
System: "Approve / Modify / Reject / Generate another"
You: "approve"
System: "What time to schedule?"
You: "8am on Eid morning"
System: "Scheduled ✓ — All pages and Instagram, Eid 8am
Setup saved for next Eid."
```

---

**Scenario 2 — Festival handled before (profile exists):**

```
Telegram: "Eid ul-Fitr is in 2 days.

Last Eid setup:
  Post type: Greeting
  Message: Warm wishes, simple and elegant
  Style: Text-based, gold and green
  Reference: [description of last reference]
  Platforms: Page X, Page Y, Instagram A
  Time: 8am

Use same setup, update something, or start fresh?"

You: "update — new reference image this year"
System: "Send the new reference image."
You: [sends new image]
System: "Anything else to change?"
You: "no"
System: "Generating fresh image with updated reference."
→ One image generated → sent for review
System: "Approve / Modify / Reject / Generate another"
You: "approve"
System: "Schedule time?"
You: "same as last year — 8am"
System: "Scheduled ✓ — Eid morning 8am
Profile updated with new reference image."
```

Even when nothing changes — a new image is always generated. Never reuses last year's image.

---

#### Special Days Calendar (Web App)

Full-year visual timeline of all special days — managed from one place. Each business manages its own profiles and post status independently using the shared calendar.

**View options:**
- Year view — all 12 months at a glance, every special day marked
- Month view — detailed view of one month
- List view — all days sorted by date with full details

**Colour-coded categories:**
```
Bangladesh National Days  — Independence Day, Victory Day, Language Martyrs Day, etc.
Islamic Calendar          — Eid ul-Fitr, Eid ul-Adha, Ramadan, Shab-e-Barat, etc.
International Days        — Mother's Day, Father's Day, World Environment Day, etc.
Commercial / Seasonal     — Valentine's Day, New Year, Black Friday, etc.
Custom                    — days you add yourself
```

**Each day on the calendar shows:**
- Day name, date, category badge
- Profile status: No profile / Profile exists / Updated this year
- Post status: Not planned / Scheduled / Posted
- Reminder status: set / not set / disabled

**Clicking any day opens:**
```
Eid ul-Fitr — April 10, 2026
Category: Islamic
Profile: Exists (last updated 2025)
Post history: Posted 2024, 2025
Reminder: 2 days before
Upcoming: 6 days away

Actions:
  View / edit profile
  View past posts
  Change reminder timing
  Start planning now (triggers Telegram conversation immediately)
  Disable for this year (skips without deleting)
  Mark as not relevant for this business
```

**Managing from the calendar:**
- Add custom special day — name, date (one-time or annual), category, advance reminder
- Edit reminder timing per day
- Disable any day for this year without permanent deletion
- Trigger planning conversation manually without waiting for automatic reminder
- Mark days as not relevant per business — stays in calendar, never triggers for that business

**Islamic calendar:**
Dates updated automatically at the start of each year based on expected lunar calendar. Note shown that exact dates may shift by 1 day based on moon sighting.

**Multiple businesses:**
Shared calendar across all businesses. Each business has its own profile, post status, and relevance settings per day.

---

#### Brand Identity

Optional per-business branding settings. Never applied automatically — only when you choose to. No AI involved — pure image processing by the system.

```
Business Brand Identity (configured in business settings)
  ├── Logo
  │     ├── File (uploaded once per business)
  │     ├── Default position (top-left / top-right / bottom-left / bottom-right / custom)
  │     ├── Default size (% of image width)
  │     └── Default opacity
  │
  ├── Watermark
  │     ├── Type: image file OR text
  │     ├── Default position
  │     ├── Default size
  │     └── Default opacity
  │
  ├── Presets — multiple named presets for different contexts
  │     ├── "Instagram post" → logo bottom-right, no watermark
  │     ├── "Facebook post" → logo top-left, light watermark center
  │     └── "Product image" → watermark diagonal center
  │
  └── Apply by default: On / Off (default: Off)
```

**Applying branding:**
- Off by default — branding step skipped unless you request it
- Turn on "apply by default" at business level to apply automatically to all new assets
- Always overridable per image — apply / skip / change preset / adjust position

```
"generate a product banner for Samsung TV with logo"
→ Generates → logo applied using default preset → preview shown

"generate a product banner for Samsung TV"
→ Generates → no branding → preview shown
```

Same applies to uploaded images — apply branding optionally before saving to library.

---

#### Asset Library View (web app)

Per product or category:
- New assets (never used)
- Scheduled (assigned to upcoming slots)
- Recently posted (last 30 days) with performance data
- Full history — all assets with usage count and performance
- Recycling status (on/off) per asset if needed

---

#### Client Reviews Management

Reviews are a dedicated asset type inside each product. They are collected, approved, and then used to generate review posts — one of the most effective forms of social proof content.

**Review asset types:**
| Type | Description |
|---|---|
| Text review | Written quote from a customer — typed or copy-pasted |
| Screenshot | Photo of Telegram chat, Facebook comment, Messenger, email, or any other source |
| Video testimonial | Short video of a customer talking about the product |

**Adding a review:**

Web app:
- Open any product → Reviews tab → Add Review
- Fill in: quote text / upload screenshot or video, reviewer name or alias (optional), star rating (optional), source (Telegram / Facebook / Messenger / Email / Other)
- Privacy setting: show name / hide name / use alias
- Save → status is Pending Approval by default

Telegram:
- Forward a screenshot of the review and say which product it belongs to
- Or type the review text and say which product it is for
- System saves it and asks: "Use reviewer's name or keep anonymous?"
- Saved as Pending Approval — you approve from web app or reply "approve"

**Review approval flow:**
Every review must be explicitly approved before it can be used in posts. This ensures no unverified or private conversation screenshot goes public accidentally.

```
Status: Pending Approval → Approved → Used / Archived
                         → Rejected (removed from library)
```

Web app shows a Review Approval Queue — all pending reviews in one place across all products.

**Using reviews in posts:**

Once approved, a review can be used in two ways:

1. **As a designed review image** — using the "Review / testimonial" image type in Image Generation. The review text, reviewer name (or alias), star rating, and product branding are composed into a shareable visual. Style is configurable (card design, background, font).

2. **As a screenshot post** — the screenshot itself is posted as-is (or lightly cropped). No design work needed. Fastest way to share authentic proof.

**Review post scheduling:**
- Review posts can be added to any slot like any other asset
- Can be recycled — same review posted again with a fresh caption
- Review recycling is off by default — turn on per product if needed
- System can suggest: "Product X has 5 unused approved reviews — want to schedule some?" when the queue is low

**Web app — Reviews section per product:**
- All reviews with status badges (Pending / Approved / Used / Archived)
- Approval queue highlighted at the top
- Filter by type, status, source
- Quick action: Approve / Reject / Generate Post / Schedule

---

### Skipped for Now — Comment & Engagement Management

Not being built in the current version. Planned as a future feature.

**What this will cover when built:**
- Reading comments on your own posts (Facebook Pages, Instagram, TikTok, YouTube)
- AI-suggested replies — you review and send manually, never auto-sent
- Comment moderation — hide/delete spam or negative comments
- Inbox management — Facebook and Instagram DMs in one place
- Keyword alerts — notify when specific words appear in comments (e.g. "price?", "available?")
- Engagement tracking — who is engaging most, what content drives comments

---

### Feature 2 — Competitor Monitoring & Analysis + Product/Content Finding

### Part A: Competitor Monitoring

#### Platforms Monitored
- Facebook Pages (competitor pages)
- Facebook Groups (competitor or niche groups)
- Instagram accounts
- YouTube channels
- TikTok accounts
- Websites (competitor websites / landing pages)

#### What Gets Tracked

**Organic activity:**
- New posts (content, format, frequency)
- Engagement metrics (likes, comments, shares, views)
- Posting patterns (what time, how often, which days)
- New products or offers announced
- Website changes (new pages, price changes, new products listed)
- Viral content (posts with unusually high engagement)

**Ad activity (Meta Ad Library — free, public, official):**
- All active Facebook and Instagram ads a competitor is currently running
- What product or offer each ad promotes
- What message and creative angle they are using
- How long each ad has been running — long-running ads = proven winners
- When a new ad campaign launches — early signal of a new product push
- TikTok Ads Library checked weekly for TikTok-specific campaigns
- Google Ads Transparency checked weekly for search ad activity

**Why ad tracking matters:**
Competitors spend real money on ads only when something is working. A competitor running the same Facebook ad for 3 months has found a winning product and message. That is the clearest possible signal of proven demand and effective positioning — far more reliable than organic posts alone.

#### How It Works (Tech)
- Facebook/Instagram organic: Meta Graph API (public page data)
- Facebook/Instagram ads: Meta Ad Library API (free, official, no scraping needed)
- YouTube: YouTube Data API v3
- TikTok organic + ads: TikTok API + TikTok Ads Library
- Websites: Puppeteer scraping on schedule
- Facebook Groups: Puppeteer (no API for group content)
- Google Ads: Google Ads Transparency Center (public)
- Runs on schedule (every 6 hours for organic, daily for ads)
- Changes detected → alert sent via Telegram

#### Alerts & Reports
- Instant Telegram alert when competitor posts something viral
- Instant Telegram alert when competitor launches a new ad campaign
- Daily digest: summary of all competitor organic activity
- Weekly ad intelligence report: all active competitor ads, what is working, how long running
- Weekly analysis: trends, patterns, what is working for competitors both organically and in ads
- Web app dashboard: full competitor timeline, analytics, and ad library

### Part B: Product Finding (Trend-Based)

#### How It Works
- User uploads their existing product catalog to the app
- System monitors: Pinterest, Facebook, YouTube, Google Trends, TikTok
- Claude analyzes what's trending in the user's product niche
- Compares with existing catalog → identifies gaps and opportunities
- Suggests new products that are: trending, attractive, high-engagement

#### Product Suggestion Criteria
- Currently trending (high search/engagement volume)
- Visually attractive / high shareability
- Complementary to existing product catalog
- Seasonal or event-driven opportunities

#### Output
- Telegram: "3 new trending products found this week — check the app"
- Web app: full product suggestions with images, trend data, source links
- Option to save suggestion to a shortlist for later action

### Part C: AI Growth Assistant

Analyses your own posting performance and gives actionable suggestions. No auto-changes — recommendations only. You decide what to act on.

#### What It Analyses

**Post performance:**
- Reach, engagement (likes, comments, shares, saves), and views per post
- Best and worst performing posts per platform per month
- Which content types perform best — product videos vs images vs reviews vs fun posts vs educational
- Which products generate the most engagement
- Performance trends — is your reach growing, stable, or declining?

**Posting patterns:**
- Your actual posting consistency vs your defined schedule
- Best times to post based on your own engagement history per platform
- Days and times when your audience is most active

**Audience growth:**
- Follower/page like growth per platform per week and month
- Which posts coincided with follower spikes

**Content gaps:**
- Platforms or accounts that have gone quiet
- Products that have high engagement but low posting frequency
- Categories being underused

#### What It Suggests

- Best times to post per platform (based on your data, not generic advice)
- Content types to increase or reduce
- Products that deserve more posts this week
- Accounts that need attention
- Content ideas when you ask ("what should I post this week?")

All suggestions are recommendations. Nothing changes automatically.

#### Reporting

- **Weekly Telegram summary** — short digest per business: top performing post, follower change, consistency score, one key suggestion
- **Monthly web app report** — full breakdown per platform: performance charts, best/worst content, audience growth, actionable recommendations
- **Ask anything via Telegram** — conversational access to your data at any time:
  - "How is my Facebook page doing?"
  - "What was my best post last month?"
  - "Which product is getting the most engagement?"
  - "Why is Instagram performing poorly?"
  - "What should I post more of?"

#### What It Does NOT Do
- Does not change your schedule automatically
- Does not post anything
- Does not make decisions — only informs yours

---

### Telegram Interaction

All via natural language:
- "Add this Facebook page as a competitor" (paste URL)
- "What did my competitors post today?"
- "Any viral posts from competitors?"
- "What trending products did you find this week?"
- "Show me my shortlisted products"
- "How is my Instagram doing this month?"
- "What was my best post last week?"

### Web UI
- Competitor list manager (add/remove/categorize)
- Per-competitor timeline and analytics
- Product catalog upload and management
- Trending product suggestions feed
- Shortlisted products board
- Growth dashboard — performance charts, posting consistency, audience trends per platform
- Monthly growth reports per business

---

## Application 2 — AI Content Creator

### Goal
Automatically research, produce, and publish original short-form video content for multiple interest-based channels across YouTube, Facebook, TikTok, and Instagram. Style: Zack D Films — short, animated, fact-based, fast-paced videos. Niches: science, math, tech, programming, and others.

AI does all the heavy work at every step. You stay involved through Telegram at each stage — giving quick feedback, approvals, or small directions. Minimal action from you, maximum awareness. Nothing moves to the next step without your signal.

---

### Channels & Niches

Multiple independent channels, each covering a different niche. Each channel is fully configurable from the web app and can be updated at any time — settings defined here are the starting point, not a fixed blueprint.

Each channel has its own:
- Idea pool, content calendar, and posting schedule
- Style profile (script tone, animation style, pacing, voice)
- Audience definition and topic boundaries
- Reference channels and videos AI learns from

**Channel config fields (all editable in the web app):**

| Field | Description |
|---|---|
| Name | Channel name |
| Niche | Broad category (e.g. Programming, Science) |
| Audience | Who watches — role, skill level, age range, interests |
| Topics to Cover | Specific topics AI should generate ideas for |
| Topics to Exclude | Topics AI must not suggest, even if related to the niche |
| Reference Channels | YouTube/TikTok/Facebook channels to learn style and ideas from |
| Reference Videos | Specific videos used as style or format benchmarks |
| Script Style | Tone, length, narration approach (e.g. fast facts, story-driven) |
| Animation Style | Visual type (e.g. motion graphics, illustrated, screen recording) |
| Voice Style | Energy level, speed, accent preference |
| Posting Platforms | Which platforms this channel posts to |
| Posting Schedule | Days and frequency |
| Language | Language of the content |
| Status | Active / Paused / Draft |

**AI respects topic boundaries.**
AI only surfaces ideas, researches, and produces content within the defined topic scope. If you want to expand or narrow the scope later, you update the channel settings — AI adjusts immediately.

Example: Programming channel is scoped to JavaScript engineering. AI will not suggest Python tutorials or DevOps content even though they fall in the same broad niche — unless you add them to the topics list.

---

### Channel Definitions

- **Programming**

  Audience:
  Topics to Cover:
  Topics to Exclude:
  Reference Channels:
    [Note: few channels name note incase get forget Fireship, ByteMonk, Transcode, LearnFree, ByteByteGo, onjsdev, The Modern Coder, CodeHead]
  Reference Videos:
  Script Style:
  Animation Style:

- **Science & Math**

  Audience:
  Topics to Cover:
  Topics to Exclude:
  Reference Channels:
  Reference Videos:
  Script Style:
  Animation Style:

- **Tech**

  Audience:
  Topics to Cover:
  Topics to Exclude:
  Reference Channels:
  Reference Videos:
  Script Style:
  Animation Style:

- **International (Geography, History, Politics)**

  Audience:
  Topics to Cover: Hiped Real News (Ex: Trump Assasination Attemp)
  Topics to Exclude:
  Reference Channels:
    [Note:few channels name note incase get forget fern]
  Reference Videos:
  Script Style:
  Animation Style:

- **Language (English, Arabic, Hindi, Bangla)**

  Audience:
  Topics to Cover:
  Topics to Exclude:
  Reference Channels:
  Reference Videos:
  Script Style:
  Animation Style:

- **Islamic (History and more)**

  Audience:
  Topics to Cover:
  Topics to Exclude:
  Reference Channels:
  Reference Videos:
  Script Style:
  Animation Style:

- **Business (Brands, Marketing and more)**

  Audience:
  Topics to Cover:
  Topics to Exclude:
  Reference Channels:
  Reference Videos:
  Script Style:
  Animation Style:

- **Health & Doctor Tips**

  Audience:
  Topics to Cover:
  Topics to Exclude:
  Reference Channels:
  Reference Videos:
  Script Style:
  Animation Style:

- **Life Hacks**

  Audience:
  Topics to Cover:
  Topics to Exclude:
  Reference Channels:
  Reference Videos:
  Script Style:
  Animation Style:

- **Fashion Tips and Tricks**

  Audience:
  Topics to Cover:
  Topics to Exclude:
  Reference Channels:
  Reference Videos:
  Script Style:
  Animation Style:

---

### Two Sources of Ideas

**Your inputs — you provide:**
- Channel or page links you find interesting or want to learn from
- Specific video links as reference or inspiration
- Topics or themes you want to explore
- Any other resources — articles, notes, anything

**AI discovery — system finds on its own:**
- Similar channels and pages in the same niche
- Trending and rising videos in the niche
- Topics gaining traction on YouTube, TikTok, Facebook

Both feed into the same idea pool. AI looks at your inputs and also surfaces similar ideas alongside them.

---

### Deep Viral Research

Before any production starts, AI studies why reference videos performed well. This is not surface-level — it is a structured research process:

**Analytics and performance signals:**
- View count, like ratio, comment volume, share count
- Estimated audience retention signals (when do viewers drop off based on comment patterns and engagement shape)
- How quickly the video gained traction — slow burn or immediate spike
- What made the algorithm push it — posting time, early engagement rate

**Content structure analysis:**
- Hook — exactly how the first 3-5 seconds are structured. What is said, what is shown, what question or tension is created
- Story structure — how the information is sequenced to keep viewers watching
- Ending — how it closes, whether it creates a loop back to the beginning or drives a strong call to action
- Pacing — how fast information moves, where pauses happen, when music or sound effects land

**Visual and production style:**
- Animation type and style — what kind of motion graphics, illustrations, or effects are used
- Text on screen — size, placement, timing, font style
- Color palette and visual tone
- Transitions between scenes

**Voice and audio:**
- Voice energy, speed, and tone
- Use of music — genre, volume level relative to voice, when it swells or drops
- Sound effects — what triggers them and why

**Frame-by-frame breakdown:**
This is a deep and resource-heavy analysis. AI will not do this automatically. Before doing a frame-by-frame breakdown of any video, AI asks your permission first — "This video looks like a strong reference. Want me to do a detailed frame-by-frame breakdown? It takes significant time." You approve it for specific videos only. When done, the breakdown becomes a reusable style reference for that channel.

All research findings are stored as a style guide per channel. Every video produced for that channel follows the learned patterns.

---

### Progressive Production Pipeline

AI works step by step. After each step it reports to you on Telegram with a short summary and asks for your signal before continuing. You respond with: ok / change this / try again / skip.

```
Step 1 — Idea Pool
AI surfaces new ideas from your inputs + its own discovery.
Telegram: "5 new ideas for Science channel. Top pick: [idea]. See full list in app."
You pick one or give a direction.

Step 2 — Research
AI runs viral research on the most relevant reference videos for the chosen idea.
Frame-by-frame only if you approved it.
Telegram: "Research done. Key findings ready in app. Proceed to script?"
You: ok / review findings first.

Step 3 — Script
Claude writes the script using the research findings — hook, story beats, ending.
Telegram: "Script ready. Review in app."
You read, leave comments or edits, then approve.

Step 4 — Visual Plan
AI plans each scene — what visuals, text on screen, timing.
Telegram: "Visual plan ready. Check in app."
You approve or request changes.

Step 5 — Image and Visual Generation
AI generates all images and graphics for each scene.
Telegram: "Visuals generated. Preview in app."
You approve or request specific changes.

Step 6 — Voiceover
AI generates voiceover from the approved script using text-to-speech.
Telegram: "Voiceover ready. Listen in app."
You approve or ask for a different tone/speed.

Step 7 — Video Assembly
AI combines visuals + voiceover + background music into a finished video.
Telegram: "Video assembled. Final preview in app."
You watch it and give final approval.

Step 8 — Post
AI posts to all configured platforms for that channel with auto-generated
title, description, hashtags, and tags.
Telegram: "Posted ✓ — Science channel, YouTube + TikTok + Facebook."
```

---

### Telegram Interaction Style
- Every update is short — one or two lines with a link to the app for details
- You never need to remember what step you are on — AI always states it
- You can pause any pipeline at any step and resume later
- You can run multiple channels in parallel — AI tracks each one separately

---

### Web UI
- Channel manager — add/remove channels, set niche, platforms, posting schedule
- Resource library — channel links, video references, topics you have submitted
- Style guide per channel — research findings, visual rules, voice tone
- Content calendar — what is in progress, queued, scheduled, published
- Production workspace — current step, script editor, visual previews, video player
- Published video history with performance metrics
- Idea pool — all pending ideas with source (your input or AI discovery)

---

## Application 3 — Reminder & Calendar

### Feature 1 — Reminder System

### Core Features
- Set reminders via Telegram or web app
- Delivered via Telegram + visible in web app dashboard
- Recurring reminders: daily, weekly, monthly, custom interval
- Priority levels: low, normal, urgent
- Smart follow-up: after delivering reminder, asks once or twice if it was acted on
- Snooze: "remind me again in 30 min / 1 hour / tomorrow"

### Telegram Flow
1. "Remind me to call Ahmed tomorrow at 6pm"
2. System confirms: "Reminder set ✓ — Call Ahmed, tomorrow 6pm"
3. At 6pm: "📌 Reminder: Call Ahmed — Done? Reply: done / snooze / skip"
4. If no reply in 15 min: one follow-up message
5. If snoozed: re-fires at the snoozed time

### Web UI
- All reminders: upcoming, recurring, completed, snoozed
- Add / edit / delete reminders
- Reminder history log

---

### Feature 2 — Calendar Manager

### Core Features
- **Two-way sync with Google Calendar**
- View, add, edit events from web app
- Morning briefing: Telegram message with today's events (sent at configurable time)
- On-demand schedule via Telegram

### Telegram Interaction

All via natural language:
- "What do I have today?"
- "What's happening this week?"
- "Add a meeting on Sunday at 3pm for supplier call"

### Google Cloud Integration
- Uses same Google Cloud project as Google Drive and YouTube
- OAuth for calendar read/write access

---

## Application 4 — Language Learning

Supports multiple languages. Each language is configured independently with its own lessons, notes, and research queue. Currently active: English, Arabic. More languages can be added at any time.

### Core Features
- Daily lessons delivered via Telegram (configurable time, default 8am)
- Lesson types rotate: vocabulary, grammar, idioms, phrasal verbs, pronunciation tips
- Lesson schedule and topics configurable from web UI

### Notes System
- Take notes via Telegram (natural language: "save this note: ..." or just send the note and say where it belongs)
- Notes saved under topics (auto-tagged or manually tagged)
- Mark notes as "research later" or "practice later"
- System sends periodic reminders for flagged notes
- Notes organized and searchable in web UI

### Web UI
- Past lessons archive
- Note editor organized by topic
- Research queue with reminder status
- Upcoming lesson preview

---

## Application 5 — Personal Knowledge & Brand Intelligence

### Concept
A personalized intelligence feed — not just news, but curated, real, verified
information across the user's interests. Delivered twice daily via Telegram.
Designed to fuel personal brand growth, content ideas, and deep knowledge.

### Interest Areas (Niches)
1. English (language, linguistics, learning resources)
2. Arabic (language, culture, resources)
3. Geopolitics — International (global events, diplomacy, conflicts)
4. Geopolitics — National (Bangladesh politics, economy, society)
5. Mathematics (discoveries, learning, interesting problems)
6. Science (latest research, discoveries, space, physics, biology)
7. Programming & Technology (new tools, frameworks, AI developments, software news)
8. Islam (scholarship, history, contemporary discussions, Islamic world events)

### Daily Delivery Schedule
- **Morning briefing** (configurable, default 8am): What's happening today / overnight
- **Evening briefing** (configurable, default 8pm): Day's top developments + content ideas

### Briefing Format (Telegram)
Each briefing includes:
- Top 2-3 items per active niche (concise bullet points)
- Source name mentioned for credibility
- 1-2 **content ideas** based on trending topics (for social media / personal brand)
- Option to reply `!more [topic]` for a deep dive on any item

### Content Idea Engine
- Based on trending topics in your niches
- Suggests post ideas, angles, formats
- "This topic is trending in tech — here's a content angle for your audience"
- Helps build consistent personal brand across platforms

### Notes & Research Queue
- `!intel save [topic]` — save an item to research later
- Saved items appear in web UI with reminder to follow up
- Connect to English Learning notes (overlapping research topics)

### Web UI
- Full briefing archive (searchable)
- Interest area preferences (add/remove/prioritize niches)
- Saved items for later research
- Content ideas board (saved ideas for future posts)
- `Tell me more` expanded view per topic

### Information Sources
- **RSS feeds** (free) — curated per niche
- Claude API summarizes RSS content into clean briefings
- RSS sources per niche:
  - English: BBC Learning English, VOA Learning English
  - Arabic: Al Jazeera Arabic, BBC Arabic
  - Geopolitics (International): Reuters, Al Jazeera English, BBC World
  - Geopolitics (Bangladesh): The Daily Star BD, bdnews24, Prothom Alo English
  - Math & Science: Science Daily, Nature, Arxiv (filtered)
  - Programming & Tech: Hacker News, TechCrunch, Dev.to
  - Islam: Muslim Matters, AboutIslam, IslamQA
  - AI: MIT Tech Review AI, The Batch (DeepLearning.AI)

---

## Application 6 — Chinese Product Finder (Bangladesh Market Focus)

### Goal
Find the best products to import from China and sell in Bangladesh.
All analysis, scoring, and recommendations are done with the Bangladesh market in mind —
buying power, culture, local trends, seasonal events, and demand patterns.

AI does not decide what to buy. AI researches, scores, and shortlists.
You make the final call quickly from a clean, prepared list.

---

### Human-in-the-Loop Flow
```
Stage 1 — AI scans all sources daily
  → Filters down to top 10-15 products
    → Each shown with: score, landed cost, local competition, BD fit
      → You mark each: Interested / Not Interested / Watch Later

Stage 2 — You trigger supplier research when ready (Telegram or web app)
  → System finds and compares top 5 suppliers
    → Telegram: short summary (name, price, MOQ, rating, factory/trader)
    → Web app: full comparison table
      → You shortlist one or two suppliers

Stage 3 — Web app only
  → Open supplier profile → read AI-drafted inquiry → edit as needed → save draft
  → When ready → you press Send explicitly
  → Nothing sent without your conscious action at that exact moment
```

---

### Sourcing Platforms (Supply Side)
Where we find the products and suppliers:
- Alibaba.com — international wholesale, verified suppliers
- AliExpress.com — smaller MOQ, good for testing products
- 1688.com — Chinese domestic, factory-direct, lowest prices

---

### The Right Mental Model for Discovery

Ecommerce platform data only tells you what is already selling. Relying only on it means you are almost always entering late. Real opportunities are found earlier — either by spotting problems before a product solution exists, or by catching products rising on supply platforms before they saturate the Bangladesh market.

The system runs two completely independent processes every day simultaneously. Both feed into the same final daily list.

---

### Process 1 — Problem to Solution (Upstream Discovery)

Start from the real world. Find problems, unmet needs, and desires in Bangladesh. Then find what product in China solves that problem. This catches opportunities before they appear on any ecommerce hot list.

```
Problem or desire found in Bangladesh conversations
        ↓
Does a product exist that solves this?
        ↓
Find it on Chinese platforms
        ↓
Is anyone in Bangladesh selling it yet?
        ↓
If not or very few → strong early opportunity
```

---

#### Category Profiles — The Foundation of Everything

All monitoring, scanning, and discovery is driven by category profiles. These are not fixed one-time settings. They are living profiles that change over time, can be complex, and each one is completely independent from the others.

---

**What a Category Profile Contains:**

```
Category: Women's Clothing
  Status: Active
  Sub-categories: casual, traditional, seasonal
  Price bands:
    Budget: 150 - 500 taka (active)
    Mid: 500 - 1500 taka (active)
    Premium: 1500 - 4000 taka (paused)
  Target margin: minimum 35%
  MOQ tolerance: max 100 units for test, max 500 for confirmed
  Preferred suppliers: factory preferred, trading ok
  Avoid: fragile items, items requiring size fitting
  Platforms to sell on: Facebook, Instagram, Daraz
  Season notes: Eid critical, monsoon slow
  Status: Active
  Created: March 2026
  Last modified: March 2026
```

Every category has its own price range, margin target, MOQ limit, preferences, and notes. Changing one category never affects another.

---

**Category Status — Can Change Anytime:**

- **Active** — full monitoring, products surface in daily list
- **Paused** — monitoring continues quietly in background but nothing surfaces to you. Use for off-season. Resume anytime — system already has weeks of trend data ready.
- **Testing** — new category at reduced intensity while you evaluate if worth pursuing
- **Archived** — monitoring stopped completely. History kept.

Pausing is not the same as removing. A paused category keeps its history and keyword intelligence intact.

---

**Multiple Price Bands Within One Category:**

Each price band is independent and can be activated or paused separately. Testing the premium market for a season — activate that band. It does not work — pause it. Other bands keep running untouched.

---

**Settings Are Versioned, Not Overwritten:**

When you change any setting the old value is not deleted. The system keeps a history:

```
Margin target — Women's Clothing:
  March 2026: 30% (initial)
  April 2026: 40% (raised — 30% not profitable enough in practice)
```

When settings change the system immediately re-evaluates:
- Watchlist products — do they still qualify?
- Today's scoring pipeline — needs re-scoring with new settings
- Keyword lists — need updating for sub-category changes?

You get a brief summary: "Settings updated. 3 watchlist products no longer meet new margin target — moved to Watch Later. 8 products previously filtered now qualify — added to tomorrow's list."

---

**Conflict Detection:**

When your settings filter out everything — no products found matching your margin target — the system tells you rather than silently showing nothing:

"No electronics products found in the last 30 days matching your 50% margin target. Current market range is 28-35%. Adjust target or keep it?"

You always know why your list is empty rather than wondering.

---

**How You Change Settings:**

From Telegram naturally — no strict syntax:
```
"pause clothing for now"
"change electronics max price to 5000 taka"
"I don't want fragile products anymore"
"add home goods category, budget range"
"stop showing me products needing big orders"
```

From web app — full category profile editor with all settings visible.

System confirms before saving any change and shows what will be affected.

---

#### Keyword Foundation — Generated Per Category Profile

Pure keyword searching is not enough. The system needs both product language and human problem language. When a new category is created the system has a short guided conversation first — either Telegram or web app — before generating anything:

```
System: "You added Women's Clothing. A few quick questions:"
"What type mainly? (casual / traditional / western / all)"
You: "casual and traditional"
"Price range?"
You: "budget and mid"
"Anything to focus on or avoid?"
You: "seasonal is important, avoid very fragile"
System: "Got it. Generating keyword lists now."
```

With those answers Claude generates lists that are specific to your exact business — not generic global lists.

**Product keywords** — specific product names, types, variations in English and Bangla relevant to your sub-categories and price bands.

**Problem keywords** — real human language describing problems those products solve, written the way Bangladeshi people actually talk. Not "affordable clothing solution" but "sasta kapor", "boro saiz pawa jay na", "bacha der eid dress koi", "halka kapor summer".

**How keyword lists evolve over time:**

- Google Trends rising queries that match your categories get added automatically
- Discovered products contribute their own language to the keyword library
- Your Not Relevant feedback pushes keywords toward lower weight
- Your Very Interesting feedback pushes keywords toward higher weight
- Monthly Claude review: which keywords produced nothing last month? Suggest removals and additions — you approve in one go
- Seasonal keywords automatically activate and deactivate based on learned patterns
- When category settings change — sub-categories added, price bands adjusted — Claude generates additional keywords for the new scope and flags any existing keywords that no longer fit

---

#### Platform Priority for Bangladesh

| Platform | Relevance | What it gives |
|---|---|---|
| Facebook groups + pages | Highest | Real BD conversations, problems, desires, buying intent |
| Facebook Marketplace BD | High | What people actively want to buy right now |
| Daraz | High | Proven BD demand, gaps in supply |
| Bikroy.com | Medium | Second-hand demand = unmet new product demand |
| Shajgoj | Medium (beauty/lifestyle) | BD-specific product desires |
| YouTube comments | High | Detailed problem expressions, wishes |
| TikTok | Medium-High | Viral trends hitting BD youth market |
| Instagram | Medium | Urban BD lifestyle and fashion trends |
| Google Trends BD | High | Search intent, rising topics, direction |
| Bangladesh news RSS | High | Events creating sudden demand windows |
| Reddit | Low-Medium | Global early signals only |
| Pinterest | Low | Global visual product trends only |
| Amazon Movers | Low-Medium | What will arrive in BD in 3-6 months |

---

#### Step 1 — Google Trends Bangladesh (Daily, Free, Zero Restrictions)

Always first. Cheapest and most reliable. No platform restrictions whatsoever.

**What it checks:**
Google Trends has several useful data points the system reads for Bangladesh specifically:

- **Interest over time** — how search volume for a keyword has moved over the past 7 days, 30 days, and 90 days. The 7-day view catches sudden spikes. The 90-day view shows whether something is a sustained trend or a one-day noise.
- **Rising queries** — Google Trends shows a "Rising" section for each topic which lists related search terms that have grown the most recently. These are often the most valuable signals — new phrases people started searching that were not common before.
- **Related topics rising** — not just the keywords you searched but what related topics are suddenly climbing. A search for "summer fan" might show "load shedding solution" as a rising related topic — a signal the system would not have caught from keyword alone.
- **Geographic breakdown** — which regions of Bangladesh are searching most. Dhaka-heavy interest is different from nationwide interest. Nationwide rising interest is a stronger signal.
- **Seasonal pattern comparison** — Google Trends lets you compare current interest against the same period in previous years. If something is rising faster this year than last year at the same time, that is a stronger signal than a normal seasonal rise.

**What counts as a signal worth flagging:**
- A keyword rising more than 20% week on week
- A keyword appearing in the Rising Queries section that was not there last week
- A related topic rising that connects to your categories that was not in the keyword list before — system adds it to the keyword list automatically for future monitoring
- A keyword showing unusual spike compared to its seasonal pattern from previous years

**What happens with flagged signals:**
Each flagged keyword or topic is noted with its direction and speed of rise. These notes are carried into later steps — when the same product or problem shows up in Facebook, YouTube, or Daraz checks, the Google Trends signal adds weight to it. A product confirmed by Google Trends plus one more source is already worth investigating.

**Cost:** Zero. Google Trends is free to access. Numbers only — no Claude involved.

---

#### Step 2 — Bangladesh News RSS Feeds (Daily, Free, Zero Restrictions)

RSS feeds are the cleanest and most reliable data source in the entire system. Bangladesh's major news outlets all provide RSS — structured, updated throughout the day, no scraping, no rate limits, no restrictions.

**RSS sources monitored:**
- The Daily Star (English) — general news, business, lifestyle
- Prothom Alo (English edition) — national news, culture
- bdnews24 — breaking news, business
- The Business Standard Bangladesh — business and economy focus
- Dhaka Tribune — national and international news
- New Age Bangladesh — alternative perspectives, business
- Channel i / NTV online feeds — broadcast news with online RSS

**What the system reads from each feed:**
RSS gives headline, publication time, and a short summary (usually 1-3 sentences). The system reads only these — never the full article. This is enough to detect relevance.

**How matching works:**
Each incoming headline and summary is checked against your problem keyword list using simple text matching — no Claude needed. If a keyword or close variation appears, the article is flagged.

**Types of signals that get flagged:**

Seasonal and weather events:
"Temperature to reach 38°C across Bangladesh this week" → cooling products, outdoor gear
"Heavy monsoon expected two weeks early this year" → rainwear, waterproof products, home drainage

Policy and regulatory changes:
"Government announces school reopening schedule" → bags, stationery, uniforms
"New import duty on electronics announced" → source electronics from China before duty kicks in
"EV motorcycle registration simplified" → accessories, charging equipment

Cultural and religious events:
"Eid ul-Fitr moon sighted — celebrations begin tomorrow" → immediate seasonal flag
"Ramadan shopping season begins — markets report surge" → food, clothing, gifting categories
"Pohela Boishakh events confirmed across Dhaka" → fashion, accessories, cultural products

Economic signals:
"Disposable income rising in tier-2 cities" → expansion of market beyond Dhaka
"Dollar rate stabilises — import costs drop" → better margins for importers right now
"Remittance inflow hits record high" → increased consumer spending likely

Business and market news:
"Daraz reports 40% growth in electronics category this quarter" → strong category signal
"Chinese import shipments delayed at Chittagong port" → supply constraint, prices may rise, act now
"Local manufacturers unable to meet demand for X" → import opportunity confirmed

**What happens with flagged articles:**
Each flagged article gets a category tag (which of your product categories it relates to), a signal type (seasonal, policy, economic, cultural), and an urgency level (time-sensitive like weather vs slower-moving like policy).

Time-sensitive signals — weather, Eid confirmed, port delays — get an immediate Telegram alert to you rather than waiting for the daily summary.

Slower signals feed into the daily analysis and add weight when the same theme appears in other sources.

**Cost:** Zero. RSS is free. Simple text matching — no Claude involved unless a cluster of signals warrants interpretation.

---

#### How Platform Monitoring Actually Works — Continuous Natural Behaviour

There are no cycles, no fixed schedules, no intervals for platform monitoring. The system behaves like a real human user on every platform — always active throughout the day, doing small natural actions spread out with human-like timing and gaps. No platform ever sees unusual burst activity because there is none.

**The continuous queue:**
The system maintains a queue of small platform tasks running all day. Each task is one small action — search one keyword on Daraz, read one Facebook group, check comments on one YouTube video, browse Bikroy Wanted listings for one category. Tasks are pulled from the queue one at a time with natural random gaps and pauses between them — exactly like a real person switching between platforms throughout their day.

**Keyword states drive everything:**
Every keyword has a current state. The state determines how many tasks get generated for that keyword and how often. Google Trends and RSS run every morning and are the engine that moves keywords between states throughout the day.

```
Dormant   → Google Trends + RSS only. Almost no platform tasks generated.
Watching  → Small signal detected. Light tasks added to queue for Tier 2 platforms.
Active    → Multiple signals confirmed. Tasks added for both Tier 2 and Tier 3 platforms.
Hot       → Strong multi-source signal. Priority tasks, deeper reads, more platforms.
Cooling   → Signals fading. Tasks gradually reduce back toward Dormant.
```

**Spike protocol:**
When Google Trends or RSS produces an urgent signal — sudden spike or time-sensitive news — that keyword immediately jumps to Hot state and priority tasks are added to the front of the queue regardless of its previous state. Time-sensitive opportunities get immediate attention.

---

#### The Platforms and What the System Does on Each

**Daraz Bangladesh**
On Daraz the system behaves like a shopper browsing the site. It searches keywords, reads search suggestion dropdowns (what Daraz auto-suggests tells you exactly what real Bangladeshis are searching right now), checks Trending Searches sections, notes how many sellers appear for each keyword and at what prices. Each keyword search is one small task. Natural gaps between tasks. Numbers only — no deep reading unless keyword is Hot.

**Bikroy.com**
On Bikroy the system behaves like someone looking for things to buy. It searches the Wanted section for keywords, counts listings, notes what people are struggling to find. A keyword generating many Wanted listings with few available listings is a direct unmet demand signal. Each search is one task, done naturally throughout the day for Active and Hot keywords.

**Shajgoj**
Only checked for beauty and lifestyle category keywords. System visits community discussion sections, searches relevant keywords, reads visible discussion titles and top-level content. Recurring wishes and complaints noted. Passes grouped themes to Claude only when signals are strong enough to justify it. Dormant keywords never checked here.

**Facebook Marketplace Bangladesh**
System searches Marketplace Wanted listings for keywords. Counts how many people are looking for something they cannot find. Pure, explicit demand signal requiring no interpretation. Each keyword search is one natural task in the queue. Counts tracked over time — direction matters more than absolute number.

**Facebook Public Groups**
Most important platform in Bangladesh. System searches public groups using Facebook's own search for problem and product keywords. Reads the top results that appear — typically 10 to 20 posts per search. Notes recurring themes, complaints, wishes. Does not try to read entire groups. Each group search is one task. Hot keywords get multiple group searches across the day. Dormant keywords are never searched here. All done at natural human pace with gaps.

**YouTube**
Uses official YouTube API. Searches for recent videos related to keywords. For Active keywords — reads video titles and descriptions to count relevance. For Hot keywords — reads comment samples from highest-engagement videos, looks for recurring complaints, wishes, and questions. API quota managed carefully — Hot and Active keywords consume quota, Dormant keywords never touch it. Each video comment read is one task in the queue.

**TikTok**
System searches TikTok for keywords and counts video results and approximate view ranges. Numbers only for Watching and Active keywords. For Hot keywords — reads video descriptions and visible comment samples for pattern detection. Each keyword search is one natural task.

**Instagram**
Hashtag post counts for keywords. Direction matters more than absolute number. Light touch — one task per keyword, only for Active and Hot keywords. Dormant keywords never checked here.

**Meta Ad Library Bangladesh**
Official free API. No scraping. System searches for product and problem keywords filtered to Bangladesh. Records: how many active ads exist, how long the longest-running ads have been active, what messages are being used. Long-running ads = proven demand. Zero ads = early opportunity or no demand. Each keyword search is one API call, one task. Active and Hot keywords checked here. Dormant keywords skipped.

**Reddit**
Searches relevant subreddits for keywords. Reads post titles and engagement levels. For Hot keywords — reads top comments on high-engagement posts for early global signals. Each subreddit search is one task. Only Active and Hot keywords checked here. Global early signal — what is big on Reddit today often reaches Bangladesh in months.

**Pinterest**
Checks pin and save counts for keywords. Rising saves indicate growing desire for a product type globally. Light touch — one count check per keyword per task. Only Active and Hot keywords.

**Amazon Movers and Shakers**
Checked once per day as a whole — reads the full Movers list and cross-references against all Active and Hot keywords in the database. Any match flagged immediately. Single daily task regardless of keyword count — efficient.

**TikTok Ads Library and Google Ads Transparency**
Checked for Active and Hot keywords only. Numbers — how many ads, how long running. One task per keyword.

---

#### Step 3 — Pattern Recognition and Claude Analysis

After tasks have run throughout the day, the system has collected signals across multiple platforms for multiple keywords. Before sending anything to Claude, it groups signals by theme automatically using simple text matching — no AI needed for grouping.

Only grouped themes go to Claude — not raw individual data points. Claude's job is narrow: look at these grouped themes, identify which ones suggest a specific unmet product need relevant to Bangladesh, describe the opportunity as a problem statement.

Short focused prompt. Low token cost. Claude returns a short list of identified problem opportunities.

---

#### Step 4 — Finding the Product in China

For each identified problem opportunity, the system searches Chinese platforms specifically for products that solve that problem. Targeted search using problem-derived keywords. Not browsing hot lists — actively hunting for the solution.

Checks order counts, seller counts, price ranges. Same number-based filter as Process 2. Products that qualify enter the scoring pipeline labelled as Problem Match.

---

#### Human Input That Reduces Work Over Time

Every time you mark a suggestion as Not Relevant, that keyword's state is pushed toward Dormant faster. When you mark something Very Interesting, that keyword's state moves toward Active faster. Over time the system naturally focuses more effort on what produces value for your specific business and wastes less time on what does not.

---

### Process 2 — Solution to Problem (Ecommerce Discovery)

Start from Chinese platforms. Find what is already rising, hot, or newly appearing. Then check if Bangladesh needs it and if the market is still open.

```
Product rising on Chinese platforms
        ↓
Check demand signals in Bangladesh
        ↓
Check local competition on Daraz
        ↓
Check global trend stage — early or late?
        ↓
Calculate real margin after landed cost
        ↓
Score and decide if worth surfacing
```

**Step 1 — Chinese platforms: hot and rising products**
Alibaba and AliExpress first. Only their own hot lists, trending sections, and movers per category — the platforms' own algorithms have already pre-filtered these. Around 50 to 200 products per category per platform. Alibaba covers wholesale trending, AliExpress covers retail trending, and together they show both sides of supply movement.

Beyond hot lists, a separate rising products scan runs simultaneously — looking at products climbing the rankings even if not yet on the hot list. A product moving from position 80 to position 20 in a category ranking over two weeks is a stronger early signal than one that has been at position 5 for months.

1688 checked last and only for products that scored well on Alibaba and AliExpress first. Chinese content means translation costs tokens. Only title and key specs translated initially. Full details only if the product still qualifies after the title check.

**Step 2 — Three-layer filter before Claude sees anything**

Layer 1 — Human configuration (free, zero AI):
Your categories, price range, minimum margin, and Not Interested history eliminate the majority instantly.

Layer 2 — Numbers filter (free, zero AI):
Orders below threshold, too many Daraz sellers, price outside range, data fingerprint unchanged since yesterday (skip and reuse old score) — eliminated here with no AI cost.

Layer 3 — Cheap Claude batch triage (one call, minimal cost):
Remaining 30 to 50 products sent to Claude in a single batched request with a short prompt. Scores returned for all of them together. Low scorers dropped. Only top candidates proceed.

**Step 3 — External demand confirmation for top candidates only**

Only products that passed all three filter layers get external checks. Not everything — just the survivors. All checks return numbers only — no text reading, no Claude involved.

Daraz Bangladesh: seller count, price range, and whether it appears in Daraz trending searches. Lightweight.

Bikroy Bangladesh: how many Wanted listings exist for this product keyword. Direct demand signal.

Google Trends Bangladesh: search volume and direction. Free, instant.

Facebook Marketplace BD: how many Wanted listings for this product keyword. Numbers only.

Meta Ad Library Bangladesh: how many active ads exist for this product keyword, how long the longest-running ad has been active. Many long-running ads = proven demand but also competition. Zero ads = early opportunity or no demand.

TikTok Bangladesh: video count and view trend for product keywords. Numbers only.

Amazon Movers: does this product appear? If yes, Bangladesh arrival window estimated.

**Step 4 — Full Claude analysis on final candidates**
After all filtering, typically 10 to 20 products remain. Full analysis happens here — Bangladesh fit, trend stage, landed cost calculation, margin estimate, discovery type assignment.

---

### How Both Processes Connect

Both processes run simultaneously every day. Their outputs are scored using the same criteria and fed into the same final daily list.

**Signal strength based on which process(es) found it:**

| Found by | Signal strength | How to treat it |
|---|---|---|
| Process 1 only | Early, unconfirmed | Very early signal — small test batch if ordering |
| Process 2 only | Confirmed supply trend | Check if you are entering early enough |
| Both Process 1 and 2 | Strongest possible signal | High confidence — act with more conviction |

A product found by both processes means real-world demand exists AND supply is already moving. That combination is rare and valuable.

**The final daily list still shows only 10 to 15 items.** Two processes do not mean twice the noise — they mean better quality and earlier discovery in the same number of daily items.

Every item is labelled clearly:
- **Trending Now** — ecommerce data confirms strong current demand
- **Early Signal** — pre-trend, one or more early signals pointing at it
- **Problem Match** — identified from a real Bangladesh problem, supply found in China
- **Problem + Trending** — found by both processes, strongest signal

---

### Efficient Cost Summary

| Work | AI Cost |
|---|---|
| Google Trends + RSS — all keywords, every morning | Zero |
| Keyword state management and queue generation | Zero |
| All platform tasks — counting, browsing, numbers collection | Zero |
| Grouping signals by theme before Claude sees them | Zero |
| Process 1 — Claude pattern analysis (grouped themes only) | Very low |
| Process 1 — targeted China search for matched problems | Zero |
| Process 2 — numbers filtering (layers 1 and 2) | Zero |
| Process 2 — data fingerprint check | Zero |
| Process 2 — batch Claude triage (one call) | Very low |
| Process 2 — 1688 title translation only | Low |
| Final full Claude analysis (10–15 products) | Moderate, justified |
| Deep investigation (only when you mark Interested) | On demand only |

Claude only does meaningful work at two points: grouped theme analysis for Process 1 (cheap, narrow prompt) and final product analysis for the shortlist (justified, high value). Everything else costs nothing.

---

### How Products Are Identified and Organised

#### The Core Problem
The same physical product exists across Alibaba, AliExpress, and 1688 with completely different names, descriptions, images, and prices depending on the seller. Even on the same platform, 30 different sellers list the same product with 30 different names. At the same time, one product often comes in multiple versions — with or without accessories, with different features or materials. These versions are related but not identical. And beyond the Chinese platforms, the same product has a presence on social media, on Amazon, on Daraz, and across the internet — all of which needs to be connected to the same product record.

Saving everything as separate products creates noise and loses the bigger picture. Merging everything into one product destroys important differences between versions. The system needs to be smarter than both extremes.

---

#### Four-Level Structure

Every product sits inside a four-level hierarchy:

```
Niche (Portable Cooling Products)
  └── Family (Neck Fan)
        └── Variant (Neck Fan with Power Bank)
              └── Listing (Seller A — Alibaba)
                  Listing (Seller B — AliExpress)
                  Listing (Factory C — 1688)
```

**Level 1 — Niche**
The broadest level. A Niche groups related product families that serve the same general market need. "Portable Cooling Products" contains Neck Fan, Portable Mini AC, Cooling Towel, Misting Fan, and Cooling Vest as separate families. The Niche level tells you when an entire market segment is heating up — not just one product but an entire space. Sometimes the Niche starts rising before any individual family becomes obviously hot. That is the earliest possible signal.

Each Niche also carries a geographic lifecycle tag per major market — where is this Niche in its trend cycle in the USA, India, and Bangladesh? A Niche that peaked in the USA 8 months ago and peaked in India 4 months ago is probably just arriving in Bangladesh now. That pattern is one of the most reliable early signals in the entire system.

**Level 2 — Family**
All versions of the same core product belong to one family. "Neck Fan" is a family regardless of whether it has LED, power bank, aromatherapy, or no extras. The family record shows the total market signal for that product — combined seller count and order count across all variants and all platforms, overall social media presence, and Bangladesh market penetration.

**Level 3 — Variant**
Specific versions within the family. "Basic Neck Fan", "Neck Fan with LED", "Neck Fan with Power Bank" are separate variants. Each has its own score, price range, margin estimate, competition level, demand data, and social media profile. This is the level where you make sourcing decisions. One variant might be oversaturated while another in the same family has barely any competition.

**Level 4 — Listing**
Individual seller listings for each variant across all platforms. Multiple sellers of the same variant shown side by side. This is where you pick your supplier.

---

#### How the System Decides What Belongs Where

**Same Niche if:** Products serve the same broad market need and the same general type of customer.

**Same Family if:** Same core function, same target customer, and any differences are add-on features rather than a fundamentally different purpose.

**Different Variant if:** Same family but meaningful physical difference — different material, added accessory, different size range, different power source — and that difference changes the price point or target buyer meaningfully.

**Separate Family if:** Different core function, different target customer, or different primary use case.

**Separate Niche if:** Serves an entirely different market need with no meaningful overlap.

---

#### How Same Products Are Matched Across Platforms

The system uses four signals together to confirm two listings are the same variant:

- **Images** — most reliable. Same physical product almost always has the same factory photos across platforms. System compares images visually.
- **Specifications** — same dimensions, weight, materials, technical features.
- **Price range** — similar price range within a reasonable margin.
- **Key features** — overlapping feature list (USB charging, 3 speeds, 360 rotation etc.)

Claude looks at all four together and makes the call — confirmed match, no match, or unsure.

**Confirmed match:** All listings attached to the same variant record. Data combined — total orders across all sellers and platforms, full price range, total seller count.

**No match:** New variant or new family created.

**Unsure:** Flagged in the web app — "These two products look similar. Are they the same variant?" One tap from you to confirm or keep separate.

---

#### Social Media Integration Into the Structure

Social media data is not just a scoring signal — it is permanently attached to each level of the structure and kept current.

**At the Niche level:**
Is this broader market segment growing as a content niche? Are more creators building audiences around it? Is there a growing community forming?

**At the Family level:**
Total TikTok videos and views featuring this product, total YouTube reviews and unboxing videos, Pinterest boards and saves, Instagram posts, Facebook Bangladesh group mentions and questions. All tracked week on week so you can see the direction — growing, stable, or fading.

**At the Variant level:**
Which specific version gets the most social attention? What are creators saying about it — praise points and complaints? What content format is working best for this product globally (short videos, tutorials, lifestyle shots)? Are Bangladeshi influencers specifically promoting it?

This social profile at the variant level serves two purposes. First it feeds into the trend score. Second it directly informs what kind of content to create when you start selling it — connecting to Module 1 and Module 7.

---

#### Deep Investigation for Interested Products

When you mark a variant as Interested, the system does not just go find suppliers. It runs a full investigation and comes back with a complete picture before you spend any money:

**Review Intelligence**
Reads all available reviews across all platforms for this variant. Pulls out the most common praise points, most common complaints, and most common use cases. Complaints are especially important — they reveal return risk, quality issues, and questions your own customers will likely ask.

**Bangladesh Market Sizing**
Estimates the potential market size in Bangladesh based on Daraz sales data, Facebook group activity, and Google search volumes. Gives you a rough sense of whether this is a niche product or a mass market opportunity.

**Local Competition Deep Dive**
Goes deeper than just counting Daraz sellers. Looks at who is selling it — established sellers with strong reviews or weak new sellers you can easily compete with. What is their pricing strategy? How quickly are their reviews coming in — are they actually selling or just listed without traction?

**Profit Scenario Modelling**
Shows multiple scenarios based on different order quantities and different selling prices. How much profit at 50 units? At 200 units? What is the break-even point?

**Risk Assessment**
Fragility risk, return rate signals from reviews, customs complexity for Bangladesh, seasonal risk if the product is time-sensitive.

**Demand Validation Suggestion**
Before ordering in bulk, the system suggests a low-cost way to test real demand — for example posting about the product on your social pages before you order. If people ask where to buy, demand is confirmed. This protects you from ordering products that look good on paper but do not actually sell.

**Content Strategy**
Based on what social media content is working for this product globally, the system suggests what kind of posts would work for your audience in Bangladesh. This feeds directly into Module 1.

All of this arrives automatically after you mark Interested. You come back to a full report, not just a supplier list.

---

#### Platform Syncing

Data across all levels and all platforms is kept current on an intelligent sync schedule based on how important and time-sensitive the data is:

**Products you marked Interested or are on your Watchlist** — synced every few hours. You are actively considering spending money on these and need current prices and availability.

**Products in your general discovery pool** — synced daily. Enough to keep trend data fresh.

**Products in Watch Later or Watching state** — synced every few days.

**Established and Fading products** — synced weekly. Just enough to catch significant changes.

**Archived products** — not synced unless activity is detected again.

**Conflict detection:** When platforms give contradictory signals about the same variant — for example Alibaba shows rising seller count but AliExpress shows falling orders — the system does not average them out. It surfaces the conflict as useful information. This could mean the product is getting easier to source but harder to sell, which is a potential saturation warning.

**Version tracking:** When a product's details change significantly — new images, substantially different description, dramatically shifted price range — the system records this as a version change rather than overwriting the old data. The history tells a story and helps you understand why something changed.

**New variant alerts:** When a new listing appears that matches an existing family but represents a new variant, the system alerts you — "A new variant of Neck Fan appeared this week — Neck Fan with Solar Charging. 8 sellers already on Alibaba. Want to investigate?" You stay informed about your families and niches evolving without going looking for changes yourself.

---

#### Why This Matters

When the same variant has 38 sellers across all three platforms with 47,000 combined orders, a strong and growing social media presence, and only 4 sellers on Daraz — that is a completely different and much more powerful picture than seeing one listing with 3,000 orders. The four-level structure with full social integration and intelligent syncing makes that complete picture visible and keeps it current.

---

### Full User Flow

**Setup (one time):** Open the web app, select your product categories, set your budget range and target profit margin. Done. System starts working automatically from that point.

**Every morning:** You get a Telegram message — "12 new product opportunities found today. Top pick: [Product Name] — 42% margin, only 4 sellers on Daraz, trending on TikTok BD. Check the app."

**Your daily review (5–10 minutes):** Open the web app, see a clean list of 10–15 products. Each shows: photo, China price → landed cost in BDT → suggested sell price → margin, score, Daraz competition, and why it was flagged. You mark each one: **Interested / Not Interested / Watch Later**. 5–10 seconds per product.

**When you mark Interested:** System runs a full deep investigation — review intelligence, Bangladesh market sizing, local competition deep dive, profit scenario modelling, risk assessment, demand validation suggestion, and content strategy. You come back to a complete report. Supplier research does not start automatically — you trigger it separately when you are ready (from Telegram or web app).

**When you trigger supplier research:** System finds top 5 suppliers. Telegram gets a short summary. Web app shows full comparison table. You shortlist. Nothing sent yet.

**When you are ready to inquire:** Web app only. Open supplier profile, read AI-drafted message, edit freely, save draft. Send only when you explicitly press Send yourself.

**Ongoing monitoring:** For every product you saved, system watches the price. Drop of 10%+ = Telegram alert. Supplier adds new products = Telegram alert. Product you watched suddenly surges on Daraz = Telegram alert.

**Seasonal planning:** 6–8 weeks before Eid, winter, Pohela Boishakh, or any major season — system sends you a Telegram heads-up with relevant product opportunities so you have time to order and receive before the peak.

**Weekly report:** Every Sunday morning — summary of the week, rising categories, your shortlisted products, price changes, and what seasons are coming up in the next few weeks.

---

### Product Lifecycle Tracking — Handling Repeated Products Smartly

A product that appeared yesterday and is still rising today is more valuable than a brand new product with one day of data. But showing the same products every day in your main list would be overwhelming and annoying. So every product goes through a lifecycle:

**New** — first time the system sees it. Appears in your daily list for review.

**Watching** — you marked it Watch Later, or it scored high but you haven't decided yet. System tracks it silently in the background. If its score rises further in the coming days or weeks, it resurfaces with a note — "This product you saw 2 weeks ago is still rising and getting stronger."

**Established** — product has been trending consistently for 3+ weeks. System sends you a dedicated one-time alert — "This product has been trending for 3 weeks straight. Strong staying power, worth serious consideration." Then stays quiet unless something changes.

**Fading** — product was trending but scores are now dropping week on week. System alerts you once — "Product you were watching is starting to decline. If you were planning to order, the window may be closing."

**Archived** — trend is dead, removed from active tracking. Stays in the database. If it comes back later, system recognises it and tells you — "This product trended before in [month] and is now showing signs of returning."

This way you never miss a product, never get flooded by the same products daily, and always know exactly where each product stands.

---

### Two Parallel Scanning Tracks

The system runs two tracks every day simultaneously:

**Track 1 — Trending Now**
Scans platform hot lists and external demand signals. These are products with proven and growing demand right now. Safe bet. Act fast before market saturates. Order confidently once validated.

**Track 2 — Early Signal (Pre-Trend)**
Looks for products that are not yet trending but showing quiet early signs. Specifically:
- Products with very low orders but unusually high Q&A activity on AliExpress — people curious but not buying yet
- Products with only 2–3 new sellers on Alibaba recently — someone is quietly testing the market
- New factory launches on 1688 being produced in volume with no orders yet — factories don't mass produce without a reason
- Products selling well in USA, UAE, or India but completely absent from Daraz Bangladesh
- A problem or topic gaining attention in Bangladesh Facebook groups that has no clear product solution yet
- Products on Amazon Movers and Shakers with zero Bangladesh presence

These products get flagged as **Early Signal** in your daily list. They are higher risk but could give you early mover advantage — entering before anyone else in Bangladesh is selling it. Recommended approach: order a small test batch first before committing.

In your daily list every product is clearly labelled — **Trending Now** or **Early Signal** — so you immediately know how to treat it.

---

### Product Discovery Types

1. **Trending Now** — already rising fast, proven demand, move before saturation
2. **Early Signal** — pre-trend, quiet signs of growth, early mover advantage
3. **Problem-Solving** — solves a real Bangladeshi problem, steady long-term demand
4. **High Profit Margin** — low China cost, high local selling price, low shipping weight

---

### Bangladesh Market Filters
Every product recommendation is filtered through:
- Is it culturally appropriate for Bangladesh?
- Is it within the buying power of the target customer segment?
- Is it importable (no ban/restriction by Bangladesh customs)?
- Is there already too much competition locally?
- Does it fit upcoming Bangladeshi events/seasons (Eid, winter, monsoon, etc.)?

### Cost Estimation and Ranking

Every product is ranked using estimated costs based on weight and size. These are standard estimates — good enough for comparing and ranking products against each other. Always shown with a note to verify actual costs before ordering.

**Weight and size drive the estimate:**

| Weight per unit | Estimated shipping cost |
|---|---|
| Under 500g | 40-80 BDT |
| 500g - 2kg | 80-200 BDT |
| 2kg - 5kg | 150-400 BDT |
| Above 5kg | flagged as high shipping cost |
| Bulky dimensions | flagged, surcharge added |

**What the system shows per product:**
- China price (CNY → BDT at live rate)
- Weight and size flag (green / yellow / red)
- Estimated shipping cost per unit
- Estimated landed cost per unit
- Suggested selling price range (from Daraz check)
- Estimated margin %

Products with red weight flags (heavy or bulky) score lower in ranking regardless of other signals — because shipping cost will likely kill the margin in practice.

All cost figures are estimates for ranking purposes. Actual costs verified by you before any order is placed.

### Bangladesh Seasonal Calendar
System knows Bangladesh's buying seasons and plans ahead:

| Season / Event | Typical Product Categories |
|---|---|
| Eid ul-Fitr | Clothing, gifts, home décor, cosmetics |
| Eid ul-Adha | Kitchenware, home goods, clothing |
| Pohela Boishakh | Fashion, accessories, gifts |
| Winter (Nov–Jan) | Winter clothing, blankets, heaters |
| Monsoon (Jun–Sep) | Rainwear, waterproof items, home tools |
| Back to School (Jan, Jun) | Stationery, bags, electronics |

System automatically surfaces relevant products 6–8 weeks before each season —
enough time to order, receive, and start selling before the peak.

### Local Competition Checker
Before recommending any product, system checks Daraz Bangladesh:
- How many sellers are already listing it?
- What is the current price range locally?
- Is demand rising or falling on Daraz?
- Too many sellers = skip. Few sellers + rising demand = strong opportunity.

---

### Supplier Finding — Three Separate Stages

**Stage 1 — Discovery**
Product marked Interested → deep investigation runs → supplier search does NOT start automatically. You decide when to move to Stage 2.

**Stage 2 — Supplier Research (you trigger)**
Triggered from Telegram or web app when you are ready.
- System searches Alibaba + 1688 for top 5 suppliers of that specific variant
- Compares: price, MOQ, ratings, response rate, years on platform, factory vs trading company
- Telegram: short summary of top 3-5 suppliers
- Web app: full side-by-side comparison table
- You shortlist one or two suppliers. Nothing sent yet.

**Stage 3 — Inquiry (web app only)**
- Open shortlisted supplier profile in web app
- Read AI-drafted inquiry message (covers price, MOQ, sample, shipping to Bangladesh)
- Edit freely — change anything you want
- Save as draft — can come back later
- Press Send explicitly when ready
- System never sends anything automatically

**Ongoing supplier monitoring**
- Saved suppliers watched for price changes and new product additions
- Telegram alert when something changes on a saved supplier

---

### Alerts & Reports
- Daily: top trending products with Bangladesh relevance score
- Weekly: full report — trends, opportunities, seasonal predictions
- Instant alert: product with strong BD demand signal found
- Instant alert: price drop on a product you're watching

---

### Telegram Interaction

All via natural language:
- "What products are trending for Bangladesh this week?"
- "Show me trending home goods"
- "Analyze this product for the BD market" (send product name or link)
- "Find suppliers for phone cases"
- "Send me the weekly report"

---

### Web UI
- Dashboard — today's top picks with Bangladesh relevance
- Discovery feed — all products with filters (type, category, score, price)
- Trend charts — what's rising/falling in each category
- Supplier comparison table
- Watchlist — saved products with price history
- Category manager — add/remove product categories to monitor

---

## API & Services Map

| Service | Used For | Provider | Status |
|---|---|---|---|
| Claude API | AI brain (all modules) | Anthropic | ✅ Have key |
| OpenClaw | Telegram integration | openclaw.ai | ❌ Setup needed |
| Facebook Graph API | FB Pages + Groups + Instagram posting | Meta | ❌ Setup needed |
| Meta Ad Library API | Competitor ad monitoring (free, public) | Meta | ❌ Setup needed |
| TikTok Ads Library | Competitor TikTok ad monitoring (free, public) | TikTok | ✅ No setup needed |
| Google Ads Transparency | Competitor search ad monitoring (free, public) | Google | ✅ No setup needed |
| TikTok API | TikTok posts | TikTok for Developers | ❌ Setup needed |
| YouTube Data API v3 | YouTube uploads | Google Cloud | ❌ Setup needed |
| Google Drive API | Media library | Google Cloud | ❌ Setup needed |
| Google Calendar API | Calendar sync | Google Cloud | ❌ Setup needed |
| Pippit.ai | Image generation (via Puppeteer) | pippit.ai | ❌ Account needed |
| RSS Feeds + Claude | Intelligence briefing summaries | Free | ✅ No setup needed |
| Chromium (snap) | Puppeteer browser automation | Already installed | ✅ Available |
| Alibaba API | Chinese product data | Alibaba | ❌ Setup needed |
| AliExpress Affiliate API | AliExpress product data | AliExpress | ❌ Setup needed |
| 1688 | Chinese domestic products (Puppeteer) | Puppeteer scraping | ✅ No API needed |
| Pinterest | Product trend monitoring (Puppeteer) | Puppeteer scraping | ✅ No API needed |
| Google Trends | Trend data | Free RSS/scraping | ✅ No API needed |

---

## Google Cloud Project (One project covers 3 APIs)
- YouTube Data API v3
- Google Drive API
- Google Calendar API

---

## Build Phases

- [ ] Phase 1 — Core Infrastructure
  - Docker setup (OpenClaw + Next.js app containers)
  - OpenClaw configured with Claude API key
  - Telegram bot created and connected (BotFather token)
  - Next.js app skeleton with all application routes
  - Basic Telegram ↔ Claude conversation working

- [ ] Phase 2 — Application 1: Social Media Manager
  - Feature 1 — Social Media Posting & Scheduling
    - Business profiles + platform account management (Facebook Pages, Groups, Instagram, TikTok, YouTube)
    - Google Drive media library integration
    - Product library + content category library
    - Asset management (upload, review, approval flow)
    - Client reviews management
    - Image generation via Pippit.ai + brand identity
    - Post creation flow (web app + Telegram)
    - Entity-based scheduling + condition-based slots
    - Content recycling system
    - Special days calendar + festival profiles
    - Post approval flow + notifications
    - Platform-specific posting (Facebook Pages, Groups, Instagram, TikTok, YouTube)
    - Web UI (schedule grid, asset library, approval queue, special days timeline)
  - Feature 2 — Competitor Monitoring, Product Finding & Growth (skip for now — build after Feature 1 is fully working)
    - Part A: Competitor tracker (FB Pages, Groups, IG, YT, TikTok, websites + ad libraries)
    - Part B: Trend monitoring + product/content suggestion engine
    - Part C: AI Growth Assistant (performance analysis + recommendations)
    - Web UI dashboard

- [ ] Phase 3 — Application 2: AI Content Creator
  - Channel manager + resource library
  - Idea finding (your inputs + AI discovery)
  - Viral research engine (analytics + content structure analysis)
  - Frame-by-frame video breakdown (permission-gated)
  - Style guide builder per channel
  - Script writing pipeline (Claude)
  - Visual plan + image generation
  - Voiceover generation (text-to-speech)
  - Video assembly (visuals + voiceover + music)
  - Telegram step-by-step progress flow
  - Multi-platform posting
  - Web UI production workspace + content calendar

- [ ] Phase 4 — Application 3: Reminder & Calendar
  - Feature 1 — Reminder System
    - Telegram reminder flow
    - Recurring reminders
    - Web UI
  - Feature 2 — Calendar Manager
    - Google Calendar two-way sync
    - Morning briefing
    - Telegram commands

- [ ] Phase 5 — Application 4: Language Learning
  - Multi-language support framework
  - Daily lesson delivery per language
  - Notes system
  - Research queue + reminders
  - Web UI

- [ ] Phase 6 — Application 5: Personal Knowledge & Brand Intelligence
  - Information source integration (RSS)
  - Twice-daily briefing delivery
  - Content idea engine
  - Web UI with archive + saved items

- [ ] Phase 7 — Application 6: Chinese Product Finder
  - Alibaba + AliExpress + 1688 monitoring
  - Overall + category trend tracking
  - Product discovery (trending, problem-solving, future demand, high profit)
  - Automated supplier finder
  - Web UI dashboard

- [ ] Phase 8+ — Future applications (TBD)

---

---

## Decisions Log

- 2026-03-29: Planning started, folder structure created
- 2026-03-31: All 8 modules fully detailed, architecture finalized, all decisions made
- Single app (Next.js) + OpenClaw — 2 containers total
- Ports: OpenClaw=18888, App=17777
- Media storage: Google Drive (not local disk)
- Image generation: Pippit.ai via Puppeteer browser automation
- Google Cloud project covers: YouTube + Drive + Calendar (one setup)
- No auth for now, single user
- Facebook Groups: post only, auto-fetch all groups user is admin of
- Special days: Bangladesh national days + Islamic calendar (Eid, Ramadan, etc.)
- Intelligence briefing: RSS feeds (free) + Claude API for summarization
- YouTube: upload with full metadata (title, description, thumbnail, tags)
- TikTok: video + photo both
