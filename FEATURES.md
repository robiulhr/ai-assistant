# Features

This file describes what every feature does and why it exists.
Written in plain language — no implementation details, no field names, no folder structures.
This file is always kept up to date. If a business decision changes, update this file.

For deep planning details, step-by-step flows, and design decisions → see `DESIGN_SPECS.md`
For build progress → see `TASKS.md`
For codebase navigation → see `CODEBASE.md`
For infrastructure setup → see `SETUP.md`

---

## Core Principle

**AI Prepares, Human Approves.**

The AI does all research, drafting, scheduling, and preparation. The human makes every final decision. Nothing with real-world consequences — posting, sending, ordering — ever happens without an explicit human action at that exact moment.

---

## Core Infrastructure

Shared services that all applications depend on. Built first, before any application.

**WhatsApp Interface**
The primary way to interact with the system. Every application communicates through WhatsApp using natural language — no commands or syntax to remember. The system understands intent from whatever you say and asks one short question only if something is genuinely unclear.

**Web App**
The secondary interface. Used for anything that needs a visual layout — schedules, asset libraries, dashboards, approval queues, settings. Accessible from any browser. Single user — no signup or multi-user management.

**Claude API Gateway**
All AI work across all applications goes through one central gateway. Manages model selection (Opus for complex reasoning, Sonnet for standard tasks, Haiku for simple tasks), tracks token usage per application, and enforces monthly budget limits. Alerts sent via WhatsApp when usage gets high.

**Job Queue**
All background work runs through a central queue. Three types: small continuous tasks (run throughout the day with human-like gaps), multi-step pipeline jobs (checkpointed so they can resume if interrupted), and heavy jobs (run in a separate lane so they never block lighter tasks). Missed tasks are always reported — the system never auto-decides what to do with them.

**Google Drive Service**
All permanent file storage lives on Google Drive — never on the server. The server is a temporary workspace only. Multiple Drive accounts supported. Files are automatically routed to whichever account has the most available space. Every file across every account is tracked in an internal index so everything appears in one unified view regardless of which account it physically lives on. The same folder structure is used on every account.

**MongoDB**
Single database shared by all applications. All structured data — settings, product details, captions, schedules, job queue, file index — lives here.

**Settings Store**
All configuration in one place. Global settings apply across everything. Each application also has its own settings that only affect that application. Readable and editable from the web app.

**WhatsApp Notification Service**
All outgoing WhatsApp messages from all applications go through one central service. If WhatsApp disconnects, messages queue and send when reconnected. Auto-reconnect attempted on session drop. QR rescan available from web app if auto-reconnect fails.

**System Health Monitoring**
RAM, CPU, disk, and Drive storage monitored continuously. WhatsApp alerts at configurable thresholds. Web app shows live system status. Ask "show usage" on WhatsApp for an instant snapshot.

**Database Backups**
Automatic daily backups to Google Drive. Daily backups kept for 7 days, weekly for 4 weeks, monthly for 6 months. Restore triggered manually.

**Image Generation Service**
Shared service used by any application that needs AI-generated images. Currently uses Pippit.ai via browser automation (Puppeteer + Chromium). No application calls Pippit.ai directly — all go through this service. Requests queue automatically — one generation at a time. If a better tool is found later, only this service needs to change, nothing else.

---

## Application 1 — Social Media Manager

Manages all social media posting for one or more businesses. Handles content organisation, scheduling, asset management, and publishing across all platforms.

### Business Profiles
The entire application is organised around businesses. Each business is a completely separate workspace with its own products, content, schedules, and connected platform accounts. Infrastructure underneath is shared. You can switch between businesses from the web app or WhatsApp.

### Platform Accounts
Each business connects to its own set of social media accounts:
- **Facebook Pages** — multiple pages, post and manage
- **Facebook Groups** — multiple groups, post only
- **Instagram** — multiple accounts, connected via Facebook login or independently — both methods available simultaneously
- **TikTok** — single account
- **YouTube** — single account

Accounts are connected via OAuth. Each account is activated manually — nothing posts until you turn it on.

### Content Organisation
All content is organised into two completely separate tracks:

**Products track** — selling-oriented content. Each product is a complete content package containing its media, captions, templates, prompts, hashtags, posting rules, reviews, and full posting history. Products have a posting frequency that controls how often they appear in the schedule.

**Categories track** — relationship-oriented content. Non-product posts that make the page feel human — fun content, educational posts, notices, behind the scenes, motivational content, special days greetings. Each category has its own asset library.

The schedule mixes both tracks. The ratio is configurable per business.

### Asset Management
All assets — images, videos, and template files — are stored on Google Drive. Text assets — captions, prompts, hashtags — are stored in the database. Assets are managed from two entry points: the full asset library and directly from within a scheduling slot. Both write to the same storage.

Every asset has a state: New, Used, Recently Used, or Archived. The system tracks which assets have been posted and when.

**Asset Variants**
Each image and video asset can have multiple variants — same product, different version. For example: a 60-second full demo video and a 15-second hook version of the same product are two variants of the same asset. Each variant has a label, type, and optional notes so you always know which one you are picking. When reposting or recycling, the system can pick a different variant instead of reusing the same file — making reposts feel fresh without needing entirely new content.

### Client Reviews
Customer reviews are a dedicated asset type linked to specific products. Reviews can be text quotes, screenshots, or video testimonials. Every review must be explicitly approved before it can be used in posts — nothing goes public accidentally. Approved reviews can be turned into designed review images or posted as screenshots directly.

### Image Generation
AI-generated images via Pippit.ai. One image generated at a time — you review, then decide to approve, modify, or generate another. Images are purpose-typed (product, lifestyle, review/testimonial, festival, fun, notice, educational) with style as a separate input. Every generation requires a prompt. Brand identity (logo, watermark) can be applied optionally after generation.

### Scheduling System
Every platform entity — each Facebook Page, each Group, each Instagram account, TikTok, YouTube — has its own completely independent schedule. Each entity has its own slots, its own queue, and its own recycling settings.

A slot is a recurring time in the schedule. Each slot has a set of condition-based rules evaluated top to bottom — the first matching rule wins. This makes slots dynamic — the same slot can post different content depending on the day, season, or what's in the queue. If no condition matches, a default rule runs.

Slots can be linked across multiple entities for cross-posting — define once, applies to all linked entities simultaneously.

### Post Approval
By default every post requires your explicit approval before going out. Configurable — can be turned off per entity, per slot, or per content type. When approval is required, a preview is sent via WhatsApp or shown in the web app. If you do not respond within 2 hours, one reminder is sent. If still no response, the slot is skipped — nothing posts without your approval.

### Content Recycling
When a slot's queue runs low, assets can be recycled — the same image or video used again but with a freshly generated caption. Never the exact same post twice. Recycling is off by default and configurable at four levels: business, slot, product, and category.

### Milestone-Based Actions
When a published post hits a performance milestone — a defined number of likes, views, comments, or shares — the system prepares an action and asks for your approval. Nothing triggers automatically without your confirmation.

Two actions are available: repost the product using a different asset variant (with explicitly defined changes to caption, title, thumbnail, or other details per platform), or increase the posting frequency for that product temporarily.

Rules are configured at three levels — business (default for all products), product (overrides business defaults), and per post (one-time override). Most products follow the business default. Products that need different treatment get their own rules. Rules are set from the web app or WhatsApp in natural language.

### Special Days
A full-year calendar of special days — Bangladesh national days, Islamic calendar, international days, and any custom days you add. Visible as a colour-coded timeline in the web app. For each special day you define how far in advance you want a preparation reminder. When the reminder fires, the system starts a planning conversation via WhatsApp — you decide what to post, generate a fresh image, and approve it. Festival profiles store the setup recipe only — a fresh image is always generated, never reused from a previous year.

### Post Creation
Posts can be created from the web app (full editor) or WhatsApp (conversational flow). The web app handles complex posts — multiple platforms, video handling, YouTube metadata. WhatsApp handles quick posts. YouTube always requires the full web app flow — title, description, thumbnail, and visibility are all required.

### Platform-Specific Behaviour
Each platform has its own post types, media requirements, and settings enforced by the system. The system automatically checks media compatibility — aspect ratios, video lengths, file sizes — and flags issues before scheduling. Auto-crop is offered when a ratio mismatch is detected.

---

### Feature 2 — Competitor Monitoring, Product Finding & Growth

**Skipped for now. Will be planned in detail when Feature 1 is fully working.**

**Part A — Competitor Monitoring**
Tracks competitor activity across Facebook, Instagram, YouTube, TikTok, and websites. Monitors both organic posts and ad activity (Meta Ad Library, TikTok Ads Library, Google Ads Transparency — all free and public). Alerts on viral posts and new ad campaigns. A competitor running the same ad for months is a proven winner — the clearest signal of demand.

**Part B — Product & Content Finding**
Monitors trend sources and your own product niche. Accepts both AI-discovered trends and human-provided resources (competitor links, topics, videos, articles). Suggests new products and content ideas. Shortlist products for later action.

**Part C — Growth Assistant**
Analyses your own posting performance — reach, engagement, best/worst content, posting consistency, audience growth. Suggests best times to post, which content to do more of, which products need attention. Weekly WhatsApp summary. Monthly web app report. Ask anything conversationally. Never auto-changes anything — recommendations only.

---

## Application 2 — AI Content Creator

Automatically researches, produces, and publishes original short-form video content for interest-based channels across YouTube, Facebook, TikTok, and Instagram. Style: short, animated, fact-based, fast-paced (Zack D Films style). Niches: science, math, tech, programming, and others.

Each channel has its own style guide, tone, and content strategy. You stay involved at every stage through WhatsApp — giving quick approvals or feedback. Nothing moves to the next step without your signal.

**Idea finding — two sources:**
- AI monitors sources (RSS, YouTube trending, academic feeds, news) and surfaces ideas
- You provide resources directly (competitor channel links, topic words, video links, articles) — AI researches from what you give it

**Production pipeline:**
Research → Script → Visuals → Voiceover → Video assembly → Review → Publish

Each stage requires your approval before the next begins.

---

## Application 3 — Reminder & Calendar

### Feature 1 — Reminder System
Set reminders via WhatsApp in natural language. One-time or recurring. Reminder delivered via WhatsApp at the right time. Manage and view all reminders from web app.

### Feature 2 — Calendar Manager
Two-way sync with Google Calendar. View, add, and edit events from the web app. Morning briefing sent via WhatsApp at a configurable time with the day's events. Ask about your schedule anytime via WhatsApp.

---

## Application 4 — Language Learning

Supports multiple languages — each configured independently with its own lessons, notes, and schedule. Currently active: English and Arabic. More languages can be added at any time.

Daily lessons delivered via WhatsApp at a configurable time. Lesson types rotate: vocabulary, grammar, idioms, phrasal verbs, pronunciation tips. Take notes during lessons via WhatsApp — saved and organised by topic. Flag notes for research or practice — system sends periodic reminders for flagged items. Full lesson archive and note editor in web app.

---

## Application 5 — Personal Knowledge & Brand Intelligence

Monitors information sources across your interest areas and delivers curated briefings twice a day via WhatsApp. Sources include RSS feeds, YouTube channels, and newsletters across topics like science, tech, Islam, AI, and others.

Briefings are summaries — not full articles. Saves interesting items for later reading in the web app. Content ideas surfaced from briefings can be forwarded directly to the AI Content Creator.

---

## Application 6 — Chinese Product Finder (Bangladesh Market Focus)

Finds the best products to import from China and sell in Bangladesh. All analysis is done with the Bangladesh market in mind — buying power, culture, local trends, seasonal events, demand patterns.

**AI does not decide what to buy. AI researches, scores, and shortlists. You make the final call.**

Daily review of top product opportunities — each shown with score, estimated landed cost, local competition, and Bangladesh market fit. You mark each: Interested / Not Interested / Watch Later. Takes 5–10 minutes per day.

When you mark a product as Interested, a full deep investigation runs automatically — review intelligence, Bangladesh market sizing, competition analysis, profit scenarios, risk assessment. Supplier research does not start automatically — you trigger it separately when ready.

Supplier inquiry drafted by AI, reviewed and sent by you manually. Never auto-sent.

Simple shipping estimate based on weight range only — no complex freight calculations.

---

## Skipped Features (Planned for Future)

**Comment & Engagement Management (Application 1)**
Reading and replying to comments, inbox management, comment moderation, keyword alerts in comments. Not built in current version.

**Application 2 Feature 2 and beyond, Applications 3–6 deep features**
Planned at high level only. Details added when ready to build.
