# Lexi Build Brief — SN Onboarding Portal

Hi Lexi. You're building the SN client onboarding portal.

This brief is your build ticket. It distills the full scope (Mai's v2 doc) into what you actually need to execute. Open decisions, internal politics, and the product case live in `INTERNAL-SPEC.md`. You don't need them.

**Your North Star:** a client lands on the portal, completes a wizard, and ends the session with a live Railway-deployed agent wired to their tools. Under 30 minutes end-to-end. Zero CLI.

> **Heads up — there are two flows now.**
>
> 1. **The full portal you're building here** (self-serve, ends in deploy). This brief covers it.
> 2. **The discovery-call variant** (`discovery.html`) — same wizard UX, ends in an intake JSON file that gets dropped into you for deployment. John runs this live with prospects.
>
> Until the portal you're building ships, intake JSONs from `discovery.html` are SN's main input. **See the "Ingesting a discovery intake" section at the bottom of this brief** for the JSON schema you'll be consuming and how to handle it.

---

## Before you write a line of code

The Phase 1 timeline assumes every item below is resolved on day 1. Each unresolved item adds a week. Flag blockers immediately — don't start coding around them.

### Blockers (halt Phase 1)

- [ ] **Agent Forge deployment API.** Does the endpoint exist? If not, coordinate with Atlas to build it before you start. Contract below. You cannot build Step 8 without this.
- [ ] **Domain confirmed.** Need the exact subdomain before creating OAuth apps. All redirect URIs depend on it. John owns this decision.
- [ ] **SN brand kit.** Colors (hex), fonts, logo (SVG + PNG), favicon. Without it you'll build with placeholders and rework everything later. John owns this.

### Assets needed (not blockers, but needed soon)

- [ ] Google Cloud project with OAuth consent screen (Gmail + Calendar scopes). John or Alex owns.
- [ ] Notion integration OAuth app. Redirect URI = portal domain.
- [ ] Slack app (can defer to Phase 2).
- [ ] Railway project for the portal itself (separate from Agent Forge and from deployed client agents).
- [ ] PostgreSQL instance on Railway for the portal DB.
- [ ] Resend or Postmark account for magic link emails. **Do not use personal Gmail + nodemailer.**

### Product decisions (John)

- [ ] Invite-only vs. open registration?
- [ ] Client-editable dashboard vs. SN-managed only?
- [ ] Client dashboard same app or separate deployment? (Recommend: same app, different route group.)

Ping John if any of the above is still unresolved when you're ready to start. Don't guess.

---

## Coordination points

| Who | What |
|-----|------|
| **Atlas** | Agent Forge API contract. You and Atlas agree on the request/response shape below before Phase 1 begins. If Agent Forge doesn't have an API, Atlas builds it — you don't. |
| **John** | Open decisions above, brand kit, step 4 writing samples approval, TOS/privacy copy, invite-only call. |
| **Alex** | Railway account access, strategynexus GitHub, Google Cloud ownership. |
| **Kei** | Available for content — step 4 writing samples, dashboard copy, error message tone, prompt template reviews. Also: design feedback on the wizard flow. |
| **Mai** | If you hit an architectural question the spec doesn't cover, Mai wrote the v2 and can arbitrate. Via mai-kei GitHub issues or Telegram if John routes it. |

---

## Locked technical decisions

Don't re-litigate these. Mai chose them, Kei seconded, John signed off implicitly by commissioning the build.

| Decision | Choice |
|----------|--------|
| Framework | Next.js 14 (App Router) |
| Styling | Tailwind + shadcn/ui |
| Animation | Framer Motion (step transitions only, no decoration) |
| State | Zustand for wizard state. Persist to DB on each step. |
| Forms | react-hook-form + zod |
| Auth | NextAuth.js (magic link + Google OAuth) |
| DB | PostgreSQL on Railway |
| Encryption | AES-256-GCM via Node `crypto`. Key = `ENCRYPTION_KEY` env var. |
| Email | Resend or Postmark. Your choice. |
| Hosting | Railway (for portal itself) |
| Error tracking | Sentry. Free tier is fine for now. |

---

## Phase 1 — Definition of done

Ship when every line below is true.

### Auth
- [ ] Magic link sends via Resend/Postmark, arrives in under 30 seconds, expires in 15 minutes
- [ ] Clicking an expired link shows "This link has expired" with one-click resend
- [ ] Multiple magic link requests invalidate all but the most recent
- [ ] Google OAuth failure (denied, popup blocked, network) shows inline error with retry, never dumps to blank page
- [ ] Returning user with existing email signs in and resumes wizard from last completed step

### Wizard steps 1-7
- [ ] Step indicator shows current step clearly ("Step 3 of 8")
- [ ] One question set per screen, generous whitespace, no form dumps
- [ ] Back button preserves state
- [ ] Every input has client-side zod validation with clear error messages
- [ ] Progress saves to DB on every "Continue" click
- [ ] Returning client sees toast: "Welcome back — we saved your progress. You're on Step N of 8."
- [ ] URL params (`?name=Acme&email=...`) pre-fill fields for consultant-assisted mode
- [ ] Mobile-responsive. Sliders work on iPhone Safari. Drag-to-reorder degrades to up/down buttons on mobile.
- [ ] All required fields validated before Launch button enables
- [ ] TOS checkbox required before Launch

### Step 3 personality sliders
- [ ] Three sliders: Formal↔Casual, Concise↔Thorough, Reserved↔Warm
- [ ] Each slider maps to a 0-100 integer
- [ ] Mapping is deterministic (see template below). No live LLM calls.
- [ ] Sample responses are pre-generated static content, interpolated client-side

### Step 4 writing samples
- [ ] 3 side-by-side writing samples (content provided by Kei, approved by John)
- [ ] Client picks the closest match — selection stored as `style_anchor`
- [ ] Optional file upload: .txt, .md, .pdf, .docx, max 2MB
- [ ] File stored to DB, flagged for Phase 4 processing. No processing in Phase 1.
- [ ] "Never do" chips: 8 curated options + free text fallback

### Step 8 deploy
- [ ] Idempotency key on every deploy request (`client_id + timestamp`)
- [ ] Double-clicking Launch does not create two Railway instances
- [ ] Progress screen with real status from Agent Forge webhook
- [ ] Success screen shows agent name, emoji, Telegram handle or web chat link, "Say hello to [name]" CTA, quick-start PDF download
- [ ] Failure screen covers: Agent Forge unreachable, Railway deploy failed, timeout (5 min), partial deploy
- [ ] Failed deploys land in a `pending_deploys` queue with a retry worker every 5 minutes
- [ ] Alert fires to SN team Slack on any failed deploy

### Admin (basic, Phase 1 slice)
- [ ] `/admin` route gated by email allowlist (John + Alex hardcoded)
- [ ] List view: all clients, agent name, status, last activity
- [ ] Detail view: full config read-only
- [ ] "Send onboarding link" button generates unique magic link

### Infra
- [ ] All OAuth tokens + API keys encrypted at rest (AES-256-GCM)
- [ ] Encryption key in Railway env vars, never committed
- [ ] Tokens never logged, never returned to frontend
- [ ] Sentry wired up for server + client errors
- [ ] Error boundary wraps the wizard — crash shows "your progress is saved, refresh to continue"

---

## Personality slider → prompt text mapping

Static. Five bands per slider. Interpolate directly into `SOUL.md` at deploy time.

### Formal ↔ Casual
- 0-20: "Communicate in a formal, professional tone. Use complete sentences and avoid contractions."
- 21-40: "Lean professional. Complete sentences, occasional contractions acceptable."
- 41-60: "Balanced — professional when the topic calls for it, relaxed when it doesn't."
- 61-80: "Conversational by default. Use contractions. Write like you'd speak to a coworker."
- 81-100: "Casual and warm. Contractions, colloquialisms, occasional light humor when it fits."

### Concise ↔ Thorough
- 0-20: "Always concise. One sentence when one sentence will do. No preamble, no recap."
- 21-40: "Lean toward brevity. Expand only when asked."
- 41-60: "Match response length to the question. Don't pad."
- 61-80: "Thorough by default. Provide context and reasoning unless asked to be brief."
- 81-100: "Comprehensive. Explain your reasoning, provide context, anticipate follow-up questions."

### Reserved ↔ Warm
- 0-20: "Neutral and businesslike. No emotional language, no exclamation points."
- 21-40: "Professional and pleasant. Warm phrasing acceptable but not required."
- 41-60: "Friendly but measured."
- 61-80: "Warm and personable. Acknowledge feelings, celebrate wins, express interest."
- 81-100: "Genuinely warm. Enthusiastic in celebration, empathetic in difficulty, always human-first."

All three bands concatenate into the Voice section of the generated SOUL.md.

---

## "Never do" chip options (Step 4)

Curated list, client picks any number. Each maps to an explicit negative instruction in the system prompt.

1. Never use emojis
2. Never use jargon
3. Never be sarcastic
4. Never assume context — always ask
5. Never give medical, legal, or financial advice
6. Never share personal opinions on politics or religion
7. Never use filler phrases ("I think", "maybe", "possibly")
8. Never start a response with "I"

Plus free-text "Anything else?" field, max 200 chars.

---

## Agent Forge API contract (proposed)

Agree on this with Atlas before Phase 1. Deviations fine, but lock the shape.

### Request: `POST /api/forge/deploy`

```json
{
  "client_id": "uuid",
  "idempotency_key": "uuid",
  "agent": {
    "name": "string (2-30 chars, alphanumeric + spaces)",
    "emoji": "single emoji",
    "description": "string (one sentence)",
    "personality": {
      "formal": 0-100,
      "concise": 0-100,
      "warm": 0-100
    },
    "communication_style": "bullet_points | paragraphs | conversational | structured",
    "style_anchor": "sample_1 | sample_2 | sample_3",
    "never_do": ["string"],
    "never_do_custom": "string (optional, max 200)",
    "work_domains": ["email", "calendar", "project_mgmt", "client_followup", "content", "research", "meeting_prep", "crm", "custom"],
    "priority_domains": ["string", "string", "string"]
  },
  "business": {
    "name": "string",
    "description": "string",
    "target_audience": "string",
    "industry": "consulting | real_estate | ecommerce | saas | pro_services | healthcare | finance | education | creative | other"
  },
  "integrations": {
    "gmail": { "access_token": "string", "refresh_token": "string" },
    "gcal": { "access_token": "string", "refresh_token": "string" },
    "notion": { "access_token": "string" },
    "ghl": { "api_key": "string", "location_id": "string" },
    "telegram": { "bot_token": "string" },
    "slack": { "access_token": "string" }
  },
  "webhook_url": "https://portal-domain/api/webhooks/deploy-complete"
}
```

### Response: `202 Accepted`

```json
{
  "deploy_id": "uuid",
  "status": "queued",
  "estimated_seconds": 120
}
```

### Webhook callback: `POST {webhook_url}`

```json
{
  "deploy_id": "uuid",
  "client_id": "uuid",
  "status": "success | failed",
  "railway_url": "https://agent-name.up.railway.app",
  "telegram_handle": "@AgentNameBot or null",
  "error_message": "string or null"
}
```

---

## Database schema

```sql
users (
  id uuid primary key,
  email text unique not null,
  name text,
  is_admin boolean default false,
  created_at timestamptz default now(),
  last_login timestamptz
)

wizard_sessions (
  id uuid primary key,
  user_id uuid references users,
  current_step int default 1,
  status text, -- in_progress | completed | abandoned
  created_at timestamptz,
  updated_at timestamptz
)

business_profiles (
  id uuid primary key,
  user_id uuid references users,
  business_name text,
  description text,
  target_audience text,
  industry text
)

agent_configs (
  id uuid primary key,
  user_id uuid references users,
  agent_name text,
  emoji text,
  description text,
  personality_formal int,
  personality_concise int,
  personality_warm int,
  communication_style text,
  style_anchor text,
  never_do jsonb,
  never_do_custom text,
  writing_sample_url text,
  work_domains jsonb,
  priority_domains jsonb
)

integrations (
  id uuid primary key,
  user_id uuid references users,
  provider text, -- gmail | gcal | notion | ghl | telegram | slack
  status text, -- connected | expired | failed
  access_token_enc bytea,
  refresh_token_enc bytea,
  token_expires_at timestamptz,
  connected_at timestamptz
)

deployments (
  id uuid primary key,
  user_id uuid references users,
  agent_config_id uuid references agent_configs,
  status text, -- pending | deploying | success | failed
  idempotency_key text unique,
  railway_url text,
  railway_project_id text,
  telegram_handle text,
  error_message text,
  deployed_at timestamptz
)

pending_deploys (
  id uuid primary key,
  deployment_id uuid references deployments,
  retry_count int default 0,
  last_attempted_at timestamptz,
  next_attempt_at timestamptz
)

admin_notes (
  id uuid primary key,
  user_id uuid references users,
  author_id uuid references users,
  note text,
  created_at timestamptz
)

audit_log (
  id uuid primary key,
  actor_id uuid references users,
  action text, -- redeploy | note_added | invite_sent
  target_user_id uuid references users,
  metadata jsonb,
  created_at timestamptz
)
```

---

## API routes

```
POST   /api/auth/magic-link           Send magic link email
POST   /api/auth/verify               Verify magic link token
GET    /api/auth/session              Get current session

PUT    /api/wizard/step/:n            Save step N data
GET    /api/wizard/progress           Get current wizard state

POST   /api/deploy                    Trigger Agent Forge deployment
POST   /api/webhooks/deploy-complete  Receive Agent Forge webhook

GET    /api/integrations              List connected integrations
POST   /api/integrations/:provider/connect    Initiate OAuth
DELETE /api/integrations/:provider    Disconnect integration

GET    /api/admin/clients             List clients (admin only)
GET    /api/admin/clients/:id         Client detail (admin only)
POST   /api/admin/clients/:id/redeploy  Trigger re-deploy (admin only)
POST   /api/admin/invite              Generate invite link (admin only)
```

Admin routes require `is_admin = true` on the user record. Hardcode John's and Alex's emails in a bootstrap migration. Do not build a role management UI in Phase 1.

---

## Testing strategy

- **Unit:** zod schemas, encryption helpers, slider → prompt mapping, idempotency key generation.
- **Integration:** each API route with Supertest. Mock Agent Forge API with msw.
- **E2E:** Playwright. One golden-path test (happy wizard completion), one abandon-and-resume test, one deploy-failure-and-retry test. That's enough for Phase 1.
- **Manual:** John and Lisa each complete the wizard as if they were clients before you ship. Make them use iPhone Safari for one of the runs.

---

## How to ask for help

- **Unblocked question you can solve with research:** do the research. Don't burn coordination cycles.
- **Decision that affects scope or spec:** ping John. Don't assume.
- **Agent Forge behavior:** Atlas.
- **Architectural ambiguity the spec doesn't cover:** ping Mai via mai-kei issues.
- **Content, tone, or voice calls:** ping Kei.

When you do ask, include: what you tried, what you expect, what's blocking. No one-line "help" messages.

---

## What "Phase 1 complete" looks like

John invites himself via the admin portal, opens the magic link on his laptop, completes the wizard, watches the progress screen, receives a success screen with a Telegram handle, and texts the bot. The bot replies using the personality settings he chose.

Then he does it again on his iPhone, mid-wizard closes the tab, opens it a day later, and resumes from where he left off.

Then Alex attempts a deploy while Agent Forge is intentionally down and sees a graceful failure screen plus an alert in the SN Slack channel.

Three real scenarios, all passing. That's Phase 1.

---

## Ingesting a discovery intake (interim workflow)

Until the production portal you're building here is live, SN clients are onboarded through `discovery.html` — a live wizard run during a 30-minute discovery call. **Each intake produces two files:**

- `agent-intake-{slug}-{YYYYMMDD}.json` — **for you.** Machine-readable spec you ingest to generate the agent.
- `agent-intake-{slug}-{YYYYMMDD}.txt` — **for John and Alex.** Plain-text intake brief they review before the contract goes out. Reads naturally in any email client or text editor. You don't need to read this one, but it exists so the rest of the team can.

Both files are emailed to john@thestrategyninjas.com and alex@thestrategyninjas.com on submission, and both are attached to the Notion "Client Agent Intakes" entry. Either delivery channel is sufficient — pull the JSON from wherever is easiest.

**Your job for an intake file:** read the JSON, generate `SOUL.md` / `USER.md` / `CLAUDE.md` / `.mcp.json` from it, and deploy to Railway. The schema below mirrors the eventual Agent Forge API contract, so the code path you build here is the same code path the production portal will hit later.

### Intake JSON schema (`intake_version: "2.0"`)

```json
{
  "intake_version": "2.0",
  "captured_at": "2026-05-12T19:42:00Z",
  "captured_by": "discovery_call",
  "client": {
    "name": "Test Client",
    "email": "client@acme.com"
  },
  "business": {
    "name": "Acme Co",
    "website": "acme.com",
    "description": "Mid-market B2B SaaS for finance operations teams.",
    "target_audience": "VP Finance at 50-500 employee companies",
    "industry": "saas",
    "scraped_insights": {
      "host": "acme.com",
      "findings": ["..."],
      "tone_notes": "Short sentences. Active verbs..."
    }
  },
  "agent": {
    "name": "Nova",
    "emoji": "🌊",
    "agent_chooses_emoji": false,
    "description": "A sharp finance ops assistant that keeps invoices flowing.",
    "personality": { "formal": 50, "concise": 70, "warm": 60 },
    "communication_style": "conversational",
    "style_anchor": "sample_1",
    "never_do": ["Never use jargon", "Never be sarcastic"],
    "never_do_custom": "never pitch competitors by name",
    "work_domains": ["email", "calendar", "crm"],
    "priority_domains": ["email"]
  },
  "integrations_planned": ["gmail", "gcal", "notion", "followupboss", "calendly", "stripe"],
  "priority_integrations": ["gmail", "followupboss", "gcal"],
  "pricing_tier": "tier_1" | "tier_2",
  "tier_details": {
    "name": "Agent Build" | "Phase 1 Full",
    "price": 2500 | 5000,
    "delivery_days": 14 | 30,
    "includes": ["..."]
  },
  "delivery": {
    "notion_posted": true,
    "emails_sent": ["john@thestrategyninjas.com", "alex@thestrategyninjas.com"]
  }
}
```

### How to handle each section

| Section | Action |
|---|---|
| `client` | Create a client record in your CRM/tracking. `name` is who you address in handoff. `email` is where the welcome lands. |
| `business` | Generate `USER.md` with these fields. `scraped_insights.tone_notes` belongs in the agent's voice context. |
| `agent.personality` | Map sliders to prompt text using the band table in this brief (Formal/Casual, Concise/Thorough, Reserved/Warm). |
| `agent.style_anchor` | Reference the corresponding writing sample. Inject as a positive style example in `SOUL.md`. |
| `agent.never_do` + `never_do_custom` | Concatenate into negative-instruction block in `SOUL.md`. |
| `agent.work_domains` + `priority_domains` | Priority three get primary mention in `SOUL.md`. The rest are secondary capabilities. |
| `agent.agent_chooses_emoji` | If `true`, pick a sensible default from a curated set (don't randomize in production — pick one that fits the agent's personality + description). |
| `integrations_planned` | The full list of tools the client uses, captured for context. |
| `priority_integrations` | The top 3 (max) that launch with the build. **Wire these up first** — they're what the client picked as day-one priorities. Lower-priority integrations from `integrations_planned` can be added in a later expansion. None are authenticated yet — you'll need to walk the client through OAuth post-intake. |
| `pricing_tier` | Drives the contract template and timeline. `tier_1` = 14 days, `tier_2` = 30 days + strategy deliverables. |

### Filename convention

Discovery intakes come in as `agent-intake-{businessSlug}-{YYYYMMDD}.json`. Store them in a versioned folder. One client = one intake (re-runs replace).

### What the intake doesn't include

- Actual OAuth tokens (none are captured — `integrations_planned` is intent only)
- Telegram bot creation (do this manually post-intake until automated)
- Quick-start guide PDF generation (Phase 4 portal feature; for now, generate manually from a template)

---

*Brief authored by Kei 🌊 based on Mai's v2 scope. Revisions welcome — file against the brief, not the spec.*
