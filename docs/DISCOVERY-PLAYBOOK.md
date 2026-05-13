# SN Discovery Call Playbook

**Owner:** John Lipe
**Tool:** `prototype/discovery.html`
**Goal:** Replace the existing pitch-deck discovery call with a live wizard that captures everything needed to deploy the client's agent. End the call with a tier decision and a delivered intake file. No second discovery call.

---

## What this is

A live, screen-shared onboarding wizard you run during a Zoom discovery call. Same UX as the productized portal, but instead of a real deploy at the end, it generates an intake (two files: one machine-readable, one human-readable) and ships it to the SN team via email + Notion. Lexi picks up the JSON and deploys; you read the plain text and run the contract.

**Why this exists:** today, discovery calls produce notes that someone has to translate into a deployment spec days later. This tool makes the discovery call itself produce the deployment-ready spec. Two artifacts from one source of truth — zero re-keying.

**Two files per intake:**
- `SNForge-{business-slug}_{agent-slug}_intake_{YYYYMMDD}.txt` — human-readable intake brief. You and Alex review this before sending the contract. It also goes into the invoice package and the Notion record.
- `SNForge-{business-slug}_{agent-slug}_intake_{YYYYMMDD}.json` — machine-readable spec. Lexi ingests this to build the agent.

Filename example: `SNForge-acme-realty_sage_intake_20260512.txt` (SNForge prefix → business → agent → "intake" → date).

---

## Pre-call setup (5 min)

1. Open `prototype/discovery.html` in a fresh browser tab (Chrome or Safari, full-screen).
2. Confirm the client's name, email, and business website ahead of time so you can pre-fill quickly.
3. Have your second monitor / second tab ready for the Notion intake DB (so you can verify the post landed).
4. If running over Zoom: screenshare the wizard tab specifically (not full screen) — keeps your notes private.

---

## During the call (~30 min)

You're driving. The client is responding. Stay conversational — don't read the wizard verbatim.

### Step 1 — Account (~1 min)

What to capture: client's name, email.

Talk track: *"Let's open up the discovery tool. First question — what's the best name and email to use for this engagement? Just so we have a record of who we're building for."*

### Step 2 — Business (~5 min)

What to capture: business name, website, description, target audience, industries (multi-select).

Talk track: *"Tell me about your business. Walk me through what you do and who you serve."* Let them talk. Type as they describe. The "website" field is important — when you continue past this step, the tool will run a quick scan and show what it found. **Don't skip pasting the website.** Industries is multi-select — pick everything that fits, and use the Other field for anything not in the list.

When the scrape animation runs, narrate: *"While we keep going, the system is pulling your homepage, your service list, your tone of voice. We'll use that as context for the agent."*

### Step 3 — Identity (~7 min)

What to capture: agent name, emoji, personality sliders, one-line description, communication style, never-do rules.

This is the biggest step. Three sections on one page:

1. **Name + emoji + description.** *"What do you want to call your agent? Pick anything. Some clients name them after the role. Some give them personality names. Up to you."* Walk them through the emoji picker — they can pick from the preset, type a custom one, or let the agent choose for them. End with the one-line description prompt: *"In one sentence, how would you describe this agent's role on your team?"*

2. **Personality sliders + live preview.** Move the sliders as you talk. The sample response below updates in real time as you slide. Make this a moment — the client sees how the tone changes with their choices. Three sliders: Formal↔Casual, Concise↔Thorough, Reserved↔Warm.

3. **Communication style + never-do.** Pick a primary style (bullets / paragraphs / conversational / structured). Then run through the never-do chips quickly — they're fast to scan. Add anything custom they mention.

### Step 4 — Work Domains (~4 min)

What to capture: 1-15 work domains selected, top 3 priority-ranked.

Talk track: *"What do you actually want this agent to do day-to-day? Pick everything that applies, then we'll rank the top three."*

The priority ranking matters — it shapes which capabilities Lexi emphasizes in the agent's system prompt. There's also an Anything-else free text field for domains we didn't list.

### Step 5 — Tools (~4 min)

What to capture: any tools they use that the agent should connect to. Then rank the top 3 — those are the ones that launch with the build.

Talk track: *"Which tools should your agent have access to? Pick everything that fits — we'll work through them all eventually. But for launch, we pick the top 3 you most want connected on day one."*

25 tools listed in a 2-column grid. Some show "SUGGESTED" — those are tools that match the work domains they picked. After they multi-select, they rank the top 3 (Agent Build tier wires up 3 integrations at launch; the rest stay in the queue). If they use something not listed (BoomTown, LionDesk, internal CRM), capture it in the "Tools we didn't list" field.

### Step 6 — Review + Confirm (~3 min)

What happens: the full agent brief renders as a 4-card summary (Agent, Business, Work domains, Launch integrations). Each card has an Edit button — clicking Edit jumps back to that step, and after fixing it the wizard auto-returns here.

Talk track: *"Here's everything we just captured. Read through it together. If anything's off, hit Edit on that section. When it looks right, we confirm."*

**Confirming auto-downloads both intake files to your machine.** Clicking "Confirm details →" runs the journey-recap animation (~4s, the same emojis they saw between steps now cycle through as the spec is "saved") and **simultaneously triggers two browser downloads**:

- `SNForge-{business-slug}_{agent-slug}_intake_{YYYYMMDD}.txt` — for you and Alex to read
- `SNForge-{business-slug}_{agent-slug}_intake_{YYYYMMDD}.json` — for Lexi to ingest

Both land in your `~/Downloads` folder. Confetti at the end. Then auto-advances to Step 7 (Pricing).

After the call (post-call section below), you'll manually forward the .txt to Alex and drop the .json into Lexi's workspace. Once the Cloudflare Worker is wired up later, this becomes automatic.

Important: the intake is captured at this step. Don't confirm until the spec is right.

### Step 7 — Pricing (~3 min)

What to capture: tier selection + "I'd like to move forward" affirmation.

Two cards side by side. Talk track: *"Two ways to start. Agent Build is the agent itself — fast, focused, deployed in two weeks. Phase 1 Full is the agent plus the strategy work behind it. Built over thirty days."*

If they're not ready: it's fine. Don't pressure. They can hit "← Back to review" to make a change, or just close the tab. The intake is already on its way to your team from Step 6, but nothing else fires until they pick a tier and confirm.

Click "Let's do it →" to advance to the final page.

### Step 8 — Our Process (~1 min)

What happens: the final page renders. No download button visible to the client. Timeline based on the tier they chose (4 weeks for Phase 1 Full, 14 days broken into 4 sprints for Agent Build) plus a "What you can expect this week" card listing the 3 things you'll send them next.

Talk track: *"And that's it. Contract from me within one business day. Once you sign, we kick off. Any questions before we wrap?"*

There's a tiny internal-only preview link at the very bottom of this page — "◧ Preview intake (internal use)" — that opens a modal showing exactly what was sent to your team. Use it post-call to verify everything captured, or to copy/download the files manually if a delivery channel fails.

---

## Post-call (5 min)

The two files dropped into your `~/Downloads` folder when you hit Confirm details on Step 6. Filenames: `SNForge-{business-slug}_{agent-slug}_intake_{YYYYMMDD}.txt` and `.json`.

1. **Read the .txt.** Open it in any text editor. It's structured like a one-page intake brief. Catch anything weird before the contract goes out.
2. **Forward the .txt to Alex.** Email or Slack it to alex@. He uses it as the contract reference.
3. **Drop the .json into Lexi's workspace.** Her preferred channel — file drop, or post in the relevant Cowork session. She doesn't need the .txt.
4. **(Optional) Post to Notion manually.** Once we set up the "Client Agent Intakes" DB, paste the .txt as a new entry and attach the .json. Skip until the DB is ready.
5. **Send the contract.** Standard SN contract + the tier they chose. Attach the `.txt` intake as a courtesy reference. One business day SLA.
6. **Follow up.** Once Lexi confirms the agent is built, schedule the handoff call.

**Fallback if downloads didn't trigger:** open the discovery tab → scroll to the very bottom of Step 8 → click "◧ Preview intake (internal use)" → use the Copy or Download button in the modal. You can re-grab either file as long as the tab is still open.

---

## Common questions clients ask

**"Is this AI doing it live?"**
Yes and no. The wizard is collecting information. The actual agent gets built by our team using your answers. The intake captures everything our deployment process needs.

**"Can I change my mind on the agent name later?"**
Yes, but it requires a redeploy. Try to land on a name you're happy with now.

**"What if I want more integrations than I picked?"**
You can add more after deployment. We pick 3-5 to start because trying to integrate everything at once delays the launch.

**"What's the difference between the two tiers, really?"**
Agent Build is just the agent — fully deployed, fully working. Phase 1 Full is the agent plus the strategic work: an AI Strategy Report for your business, a future-state org chart showing where AI fits, an AI employee strategy, and a roadmap for what comes next. If you want the agent to be the start of a bigger AI transformation, take Phase 1 Full. If you just want the agent, take Agent Build.

**"What if I'm not ready to pick a tier right now?"**
Totally fine. We can leave the wizard here, I'll send you a recap of what we captured, and we can decide on the tier when you're ready. The intake doesn't get sent until you submit.

---

## When to NOT use this tool

- The client wants a heavily custom build that doesn't fit the agent template (e.g., multi-agent system, complex workflow engine). Run a normal discovery instead.
- The client is technical and wants to spec their own system. Use a working session, not the wizard.
- You haven't done a 5-min intro on what an AI Chief of Staff is. The wizard isn't the pitch — it's the discovery. Pitch first, wizard second.

---

## What's mocked vs real

**Current state (interim, good for live discovery calls starting tomorrow):**
- ✓ Wizard captures the full intake
- ✓ Auto-downloads `.txt` + `.json` to your Downloads folder on Confirm
- ✗ Email to john@/alex@ — not wired, you forward manually after the call
- ✗ Notion post — not wired, paste manually if the DB exists

**Production wiring (Phase 2, when ready):**

Replace the `downloadIntakeFiles(data)` call in `Step6Review`'s useEffect with a `fetch()` POST to a Cloudflare Worker. The Worker:
1. Receives the intake JSON
2. Sends emails via Resend with both files attached (to john@ + alex@)
3. Posts a new entry to the Notion "Client Agent Intakes" DB
4. Returns success/failure

Estimated build: ~80 lines of code, $0/month at SN's volume. Setup needs: Cloudflare account (Workers + Pages), Resend API key, Notion integration token, Notion DB structure (Alex to define columns).

---

*Playbook authored by Kei 🌊. Update as the call flow evolves.*
