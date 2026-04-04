# Project Journey

The full lifecycle of this project — from first idea to production and beyond.
Written as things happen. Every significant decision, problem, pivot, and lesson recorded here.

This file follows the Software Development Life Cycle (SDLC):
1. Idea & Requirements
2. Planning
3. System Design & Architecture
4. Database Design
5. Implementation
6. Testing
7. Deployment
8. Maintenance & Iterations

---

## Chapter 1 — Idea & Requirements
**Period:** March 2026

### The Starting Point
The project started as a personal need — one person running a business in Bangladesh, managing social media manually, researching products from China, trying to stay on top of information across multiple areas of interest, and learning languages. All of this was happening manually, scattered across different tools, taking too much time.

The idea: build a single AI-powered assistant that handles all of this through Telegram and a web app — running on a personal VPS, fully owned, no subscriptions beyond the AI API itself.

### Original Scope — 8 Modules
The first version of requirements identified 8 separate modules:
1. Social Media Manager
2. Image Generator
3. Competitor Monitor
4. AI Content Creator
5. Chinese Product Finder
6. English Learning Helper
7. Reminder & Calendar
8. Personal Knowledge Base

### First Major Restructure
During requirements review it became clear that several modules were closely related and should live together as features of the same application rather than separate modules. Key decisions:

- Modules 1, 2, 3 (Social Media Manager + Image Generator + Competitor Monitor) → merged into **Application 1: Social Media Manager** with Image Generation as part of asset management and Competitor Monitor as Feature 2
- Module 7 (Reminder) + Module 8 (Calendar) → merged into **Application 3: Reminder & Calendar**
- Module 6 (English Learning) → expanded to multi-language → **Application 4: Language Learning**
- A new application was added that was not in the original 8: **Application 2: AI Content Creator** — for running interest-based channels on YouTube, Facebook, TikTok, Instagram

Final structure: **6 Applications** instead of 8 modules.

### Core Principles Established
Two principles were defined early that shaped every subsequent decision:

**"AI Prepares, Human Approves"** — AI does all the heavy work but never takes final action alone. Every post, message, order, and publish requires an explicit human action at that exact moment. This came from recognising that automated actions with real-world consequences (sending supplier messages, posting content, placing orders) are high-stakes and must remain fully conscious decisions.

**Natural Language Always** — The Telegram interface must always accept natural language. No command syntax to memorise. The system understands intent from whatever you say. This came from the insight that forgetting exact command syntax creates friction and makes the tool feel like a command-line interface rather than an intelligent assistant.

---

## Chapter 2 — Planning
**Period:** March – April 2026

### Approach
Planning was done at the business level first — what each feature does, how it behaves, what the rules are — before any technical decisions. The goal was to understand the product fully before thinking about implementation.

### Core Infrastructure Planning
Before planning individual applications, the shared infrastructure was defined. Key decisions:

**Technology Stack:**
- OpenClaw (port 18888) — Telegram + Claude API integration
- Next.js (port 17777) — web application
- MongoDB (port 27017, already running on VPS) — all structured data
- Google Drive — all file storage (never the server)
- Docker — both containers isolated from existing VPS applications
- Claude API — three model tiers (Opus/Sonnet/Haiku) per task complexity

**Isolation Decision:**
The VPS already runs other applications (2 Next.js e-commerce apps, 2 Express APIs, 1 React app). Docker hard caps (4GB RAM, 2 CPU cores) were chosen to ensure the AI Assistant can never starve existing applications of resources.

**Storage Decision:**
After considering server storage vs cloud storage, Google Drive was chosen for all permanent file storage. Reasons: survives VPS failure, accessible from anywhere, free 15GB per account with easy multi-account expansion. Server is temporary workspace only — files deleted immediately after Drive upload.

**Multi-Account Drive Strategy:**
A key design question was how to handle multiple Google Drive accounts as storage fills up. Three options were considered:
- Option A: Split files freely across accounts (files of same product could end up on different accounts)
- Option B: Assign products to accounts (product always on one account)
- Option C: Same folder structure on all accounts, route files by available space, track everything in an internal index

Option C was chosen — simplest for a single user who never browses Drive directly, only uses the web app. The internal file index in MongoDB means account boundaries are invisible in the web app.

**Continuous Queue (not fixed cycles):**
Platform monitoring was initially planned as fixed scheduled cycles (check every 3 days, run scan at 2am). This was rejected — fixed cycles create burst activity patterns that platforms can detect. Decision: all monitoring runs as a continuous queue of small tasks throughout the day with human-like random gaps.

**Missed Tasks Policy:**
When the system comes back online after downtime, it never auto-decides what to do with missed tasks. It always reports what was missed and presents options. The user decides. This was non-negotiable.

**Job Queue — Three Lanes:**
Background jobs were designed with three separate lanes to prevent long-running jobs from blocking quick tasks:
- Type 1: Small continuous tasks (natural human-like flow)
- Type 2: Multi-step pipeline jobs (checkpointed, resumable)
- Type 3: Heavy jobs (separate lane, never blocks Type 1/2)

**Database Backup:**
Daily backups to Google Drive. 7 daily, 4 weekly, 6 monthly. MongoDB running outside Docker so standard mongodump works directly.

### Application 1 — Social Media Manager Planning
The most detailed planning of any application. Key decisions and pivots:

**Business Profiles:**
The application was designed around businesses as the top-level entity. Each business is a completely separate workspace. Infrastructure underneath is shared.

**Two Content Tracks:**
Content was split into two tracks — Products (selling-oriented) and Categories (relationship-oriented). The ratio between them is configurable per business. This came from the insight that pages with only product posts feel like shopping catalogues — category posts create personality and audience connection.

**Asset Types Expanded:**
Initial thinking had assets as just images and videos. Planning expanded this to: images, videos, template files (Drive), captions, prompts, hashtag sets, posting rules, seasonal notes (MongoDB). A key clarification: captions and prompts are structured text data and belong in MongoDB, not Drive. Drive holds files only.

**Client Reviews as a First-Class Asset:**
Reviews were identified as missing from the initial asset structure. They are important social proof content. Added as a dedicated asset type per product — text quotes, screenshots, video testimonials — each requiring explicit approval before being usable in posts. A review/testimonial image type was added to image generation.

**Entity-Based Scheduling:**
Initial design had slots selecting which pages to post to. This was revised — each platform entity (each Facebook Page, each Group, each Instagram account, TikTok, YouTube) has its own completely independent schedule. Cross-posting handled via linked slots. This gives completely different posting strategies per entity.

**Condition-Based Dynamic Slots:**
Slots are not simple time-based triggers. Each slot has a set of rules evaluated top to bottom — first match wins. This makes slots dynamic and context-aware without needing separate slots for every scenario.

**Posting Frequency Naming:**
An attempt was made to use "Boost" for product posting priority. Rejected — sounds like paid Facebook boost. "Campaign" tried next. Also rejected. Final decision: plain language "Posting Frequency" — X times per week until date. No special term, no marketing jargon.

**Instagram Connection:**
Initially assumed Instagram always comes through Facebook login. Corrected — the user manages pages they didn't create, so the desired Instagram may not be linked to their Facebook. Fixed: both connection methods (via Facebook OR independent OAuth) always available simultaneously.

**Image Generation:**
Initially planned as a separate feature. Merged into asset management — it is one asset source among several, not a standalone feature.

**Image Types vs Styles:**
Initial design had "Cartoon" and "Text-based" as image types. Corrected — a festival post can BE cartoon or text-based — these are styles, not types. Types are purpose-based (Product, Lifestyle, Review/Testimonial, Festival, Fun, Notice, Educational). Style is a separate field in every type. Prompt is required in every type — no exceptions.

**Festival Profiles:**
A profile stores the setup recipe (colours, style, elements) — never the final image. Every year a fresh image is always generated, even with the same profile. This prevents the same greeting image going out year after year.

**Recycling — Assets Not Posts:**
Recycling reuses the asset (image or video) with a freshly generated caption. Never the exact same complete post twice. Off by default, configurable at four levels.

**Image Generation Flow:**
Initially planned to auto-generate 3 draft images when queue is low. Corrected — ask first (generate / recycle / manual / skip), never auto-generate. One image at a time, not 3 at once.

**Content Shortage Handling:**
When queue is low, system asks what to do — it never auto-decides. Options presented: generate new / enable recycling / add manually / skip slot.

**Comment Management:**
Identified as a needed feature but deliberately skipped for current version. Documented as a future feature with full scope defined so it is not forgotten.

**AI Growth Assistant:**
Initially placed as a standalone section inside Feature 1. Later correctly identified as part of Feature 2 — it belongs with competitor monitoring and product finding as Part C of the intelligence layer. All three parts feed into the same goal: making better content decisions.

### Messaging Platform Decision — Switched from WhatsApp to Telegram

Initially planned to use WhatsApp via OpenClaw with Baileys library. During pre-build research, key problems were found:
- Baileys is unofficial — violates WhatsApp ToS, real account ban risk
- QR scan gives OpenClaw linked device access to entire WhatsApp account — all chats, no permission scoping
- Session management complexity — QR rescans needed when session expires

**Decision: switched to Telegram.** Reasons: official bot API (no ban risk), bot only receives messages sent to it (no personal chat access), stateless (no session/QR scan ever), Telegram queues messages while bot is offline, rich formatting with inline buttons, free, OpenClaw supports Telegram natively.

**Video handling:** Telegram bot API has 50MB file size limit — most product videos exceed this. Videos never sent through Telegram — system sends Drive link or web app link instead. Small files (images, short clips) sent directly. This is cleaner than sending large files through a messaging app.

### Key Planning Lessons
- Fixed cycles feel like a natural starting point but are wrong for monitoring tasks — continuous queue is always the right model
- Real-world actions (send, post, order) must always be separate, explicit, conscious steps — never bundled in an automated flow
- Shipping calculation complexity was recognised early and deliberately scoped out — simple weight table only
- Planning can become endless — the rule adopted: plan a feature in detail only when about to build it
- WhatsApp unofficial libraries (Baileys, whatsapp-web.js) carry real ban risk and full account access — Telegram official bot API is always the better choice for personal assistants

### Planning File Structure
The planning process produced a very large single document. Before starting implementation it was restructured into multiple files with clear purposes:

| File | Purpose |
|---|---|
| `PLANNING.md` | Index — points to all other files |
| `FEATURES.md` | What every feature does and why — always up to date |
| `DESIGN_SPECS.md` | Full detailed planning snapshot — reference only |
| `TASKS.md` | Build progress with per-task guidance |
| `CODEBASE.md` | File and folder map — grows with the code |
| `SETUP.md` | Reusable infrastructure setup guide |
| `JOURNEY.md` | This file — full project lifecycle story |

---

## Chapter 3 — System Design & Architecture
**Period:** *(Fill in when Task 0.1 is completed)*

### Architecture Diagram
*(Add diagram description and key decisions here)*

### Key Design Decisions
*(What was decided and why)*

### Trade-offs Considered
*(What alternatives were evaluated and why they were rejected)*

---

## Chapter 4 — Database Design
**Period:** *(Fill in when Tasks 0.3 and 2.1 are completed)*

### Phase 1 Schema (Core Infrastructure)
*(Collections designed, key decisions, fields that were added or removed during design)*

### Phase 2 Schema (Social Media Manager Feature 1)
*(Collections designed, key decisions, what changed from initial thinking)*

### Schema Evolution
*(Fields added during implementation that were not in the initial design — and why)*

---

## Chapter 5 — Implementation
**Period:** *(Fill in as phases are completed)*

### Phase 1 — Core Infrastructure
*(Start date, end date, what was built, problems hit, solutions found)*

**Task 1.1 — Docker Setup**
*(What was done, any gotchas, how long it took)*

**Task 1.2 — OpenClaw + Claude**
*(What was done, any gotchas)*

**Task 1.3 — Telegram Connection**
*(What was done, any gotchas)*

*(Continue for each task...)*

### Phase 2 — Social Media Manager Feature 1
*(Fill in when Phase 2 begins)*

### Phase 3 onwards
*(Fill in as phases are completed)*

---

## Chapter 6 — Testing
**Period:** *(Fill in as testing happens)*

### What was tested and how
*(Per feature — what test scenarios were run, what broke, what was fixed)*

### Bugs found during testing
*(What broke, what caused it, how it was fixed)*

---

## Chapter 7 — Deployment
**Period:** *(Fill in when first deployment happens)*

### First Deployment
*(What was deployed, how, any issues)*

### Configuration Changes from Development to Production
*(What had to change when going live)*

---

## Chapter 8 — Maintenance & Iterations
**Period:** *(Ongoing — add entries as they happen)*

### Issues Found in Production
*(What broke in real use, how it was fixed)*

### Features Added After Launch
*(What was added, why, how it changed the original plan)*

### Performance Issues
*(What was slow, how it was diagnosed, how it was fixed)*

### Lessons Learned
*(What would be done differently if starting over)*
