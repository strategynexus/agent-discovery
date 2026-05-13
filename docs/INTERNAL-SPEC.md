# SN Agent Onboarding Portal — Internal Spec

**Owner:** John Lipe (CIO)
**Build lead:** Lexi (agent, on Alex's machine)
**Technical coordination:** Atlas (Agent Forge CTO agent) + Alex Mont-Ros
**Status:** Scope locked (Mai v2). Pending John's open decisions + Lexi pre-phase-1 blockers.
**Last updated:** 2026-04-24

> **There are now two versions of this tool.** This document is the **self-serve client portal** (v1) — the productized version that Lexi will build, ending in a real Railway deployment.
>
> A **discovery-call variant (v2)** also exists. See [`DISCOVERY-PLAYBOOK.md`](DISCOVERY-PLAYBOOK.md). Same UX, but ends in an intake JSON file (not a deploy), and is meant to be screen-shared during a 30-min discovery call. v2 is the lower-effort, near-term play. v1 is the long-term productized play.

---

## What this is

A branded web portal that onboards Strategy Ninjas clients into their own AI agent in under 30 minutes, without CLI, Node, or a manual consultant walkthrough. Replaces the current ad-hoc technical setup with a designed wizard that ends in a deployed, running agent on Railway.

Two modes, same flow:
- **Self-serve** — client receives a link, completes the wizard independently.
- **Consultant-assisted** — SN team shares the portal on a Zoom and walks the client through. Same URL, same UI, optional URL-param pre-fill to reduce friction.

By the end: client has a live agent on Railway with their personality, voice, work domains, and integrations wired in. They get a Telegram handle or web chat link and a first-message prompt.

---

## Why now

Current onboarding requires:
- A technical consultant physically walking the client through CLI + Git + Railway
- No standardized capture of voice, personality, or integration preferences
- No client-facing record of the setup
- No repeatability — every onboarding is bespoke

Result: onboarding is slow, inconsistent, and doesn't scale past the team's manual bandwidth.

The portal makes onboarding a product, not a service.

---

## Success criteria

A new SN client goes from "I want an agent" to "my agent is running and I just texted it" in under 30 minutes, including the consultant call. No CLI. No technical setup. No file creation. Just a web experience and a working agent on the other side.

Measurable:
- Median wizard completion time: < 15 minutes
- Deploy success rate: > 95% (retryable failures don't count)
- Zero manual file creation by SN team per client
- Client can reconnect an expired OAuth token without contacting SN

---

## Client journey (the experience, not the implementation)

| Step | What the client does | What we capture |
|------|---------------------|-----------------|
| 1. Account | Magic link or Google OAuth | Email, session |
| 2. Business profile | Describes their business | Name, description, target audience, industry |
| 3. Agent identity | Names agent, picks emoji, moves personality sliders | Agent name, emoji, 3 personality integers (0-100), one-sentence description |
| 4. Voice & style | Picks communication style, lists "never do" rules, optional writing sample | Style preferences, negative rules, optional tone file |
| 5. Work domains | Multi-selects what the agent is for, ranks top 3 | Domain list, priority order |
| 6. Integrations | OAuth-connects tools they selected in step 5 | Encrypted tokens for Gmail, Calendar, Notion, GHL, Telegram, Slack |
| 7. Review | Sees full summary, edits inline, accepts TOS, clicks Launch | Confirmation |
| 8. Deployment | Watches progress screen, lands on success with Telegram handle / web link | Agent deployed, accessible |

Each step is one question set per screen. No form dumps. Progress is saved on every step completion — client can close the tab and resume from any device.

---

## Roles & ownership

| Person / agent | Role |
|---------------|------|
| **John Lipe** | Product lead. Owns brand kit, writing samples (step 4), TOS/privacy, domain decisions, open questions. |
| **Alex Mont-Ros** | Technical sponsor. Owns the Agent Forge relationship on Lexi's behalf. Provides strategynexus GitHub access, Railway account, Google Cloud project. |
| **Atlas** | Agent Forge CTO agent. Must expose a clean deployment API (contract in Lexi's brief). Lexi coordinates with Atlas on the contract. |
| **Lexi** | Build lead. Executes all four phases. Owns the portal codebase, admin UI, client dashboard, deployment pipeline integration. |
| **Lisa Mont-Ros** | COO sign-off on client-facing copy and TOS. |
| **Kei** | Drafted this spec and the prototype. Available for content work (step 4 writing samples, dashboard copy, error messages) and design review. |

---

## Phases

### Phase 1 — Core wizard + deploy (MVP) — 2-3 weeks

The wizard works end-to-end and produces a live agent. No frills, no integrations, no dashboard polish.

**Includes:**
- Magic link auth
- Steps 1-7 UI
- Wizard progress persistence
- Agent Forge deploy call
- Success screen
- Basic admin client list
- Deploy failure handling + retry queue

**Excludes:** OAuth integrations (step 6 collects intent only, actual OAuth in phase 2), client dashboard, tone analysis, live personality preview.

**Add one week to the estimate for each unresolved pre-phase-1 blocker.**

### Phase 2 — OAuth integrations — 1-2 weeks

Step 6 actually works. Clients connect real tools.

- Gmail, Calendar, Notion, Slack OAuth
- GHL API key input + validation
- Telegram BotFather guided setup
- Integration health monitoring
- Reconnect flow for expired tokens

### Phase 3 — Admin portal + client dashboard — 1 week

SN team sees all clients. Clients see their own agent.

- Admin client list + detail + re-deploy
- Abandoned wizard alerts
- Deploy error queue
- Client dashboard with integration status, personality edit, re-deploy trigger
- Agent health polling

### Phase 4 — Polish & scale — ongoing

Net-new features. Not on the critical path.

- Writing sample tone analysis
- Live personality preview (sample response rendering)
- Usage analytics
- White-label option

---

## Open decisions (need John)

These block or shape the build. Listed in order of urgency.

1. **Domain.** `onboard.strategyninja.ai`, `launch.strategyninja.ai`, or something else. Blocks all OAuth app creation.
2. **Invite-only vs. open registration.** Affects the auth flow and the admin portal's invite generator. Blocks Phase 1 auth work.
3. **Brand kit.** Colors, fonts, logo, favicon. Blocks all styling work — Lexi will build against placeholders otherwise.
4. **Can clients edit settings post-onboarding, or is it SN-managed?** Affects whether the client dashboard is read-only or editable. Blocks Phase 3.
5. **Can SN automate Telegram bot creation via the Bot API, or do clients do BotFather manually?** Affects Phase 2 scope. BotFather is the single most confusing step for non-technical clients.
6. **Who writes the side-by-side writing samples in step 4?** Content work, not code. Kei can draft, John or Lisa approves.
7. **Railway compute — who pays?** Per-client Railway instances consume resources. SN-absorbed, pass-through, or priced into the engagement?
8. **Confirmation email after deploy?** Success screen only, or also an email? Defaults to: yes, send email, with Telegram handle and a quick-start guide PDF.
9. **TOS and privacy policy.** Need to exist before launch button works. Legal-owned, not Lexi-owned.
10. **Max integrations per agent?** Unlimited by default. Revisit if Agent Forge hits context or memory limits.

---

## Risks

| Risk | Mitigation |
|------|-----------|
| Agent Forge has no deployment API today | Lexi's first task is negotiating the contract with Atlas. This is a prerequisite, not a parallel track. |
| Personality sliders → prompt text mapping is hand-waved | Kei owns the template (lives in Lexi's brief). Static mapping, 5 bands per slider, no live LLM inference. |
| OAuth token expiration breaks agents silently | Integration health polling in Phase 3. Graceful degradation — expired Gmail doesn't crash the agent, it just disables that tool. |
| Client abandons wizard, no follow-up | 48-hour abandoned-wizard alert in admin portal + automated follow-up email. Phase 3. |
| Deployment timeout leaves client staring at spinner | 5-minute webhook cutoff, then "we'll email you when ready" screen. Agent Forge has 24-hour SLA on deploy-or-escalate. |
| Double-click on Launch creates two Railway instances | Idempotency key on the deploy request. Non-negotiable — Lexi implements in Phase 1. |

---

## Distribution plan

**Phase 1 goes to John and Lisa first.** They run three real client onboardings through it themselves before any actual client touches it. Anything confusing, slow, or broken gets fixed before it ships externally.

**Phase 2 invites three friendly clients** with white-glove support. Alex or John on a Zoom, using consultant-assisted mode. Measure: can a non-technical client complete the wizard without intervention.

**Phase 3 is the general launch.** Magic links go out, self-serve becomes the default, consultants join only when asked.

---

## What's next

1. John answers open decisions 1-3 (domain, invite-only, brand kit) — unblocks Lexi's Phase 1 start.
2. Lexi + Atlas agree on the Agent Forge API contract.
3. Lexi runs her pre-phase-1 checklist (see Lexi Build Brief).
4. Kei drafts step 4 writing samples for John's approval.
5. Phase 1 build begins.

---

*Spec authored by Kei 🌊 based on Mai's v2 scope doc. April 24, 2026.*
