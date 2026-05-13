# Strategy Ninjas — Agent Discovery

A live, screen-shared wizard for SN discovery calls. Captures the full agent spec in 30 minutes, auto-downloads two intake files (one human-readable, one for Lexi) on confirmation.

**Live URL:** https://strategynexus.github.io/agent-discovery/

## How to use it

1. Open the live URL in Chrome or Safari.
2. Screenshare the tab during a discovery call.
3. Walk the client through all 8 steps.
4. On Step 6 ("Confirm details"), two files drop into your `~/Downloads`:
   - `SNForge-{business}_{agent}_intake_{YYYYMMDD}.txt` — for you + Alex
   - `SNForge-{business}_{agent}_intake_{YYYYMMDD}.json` — for Lexi
5. Continue to pricing (Step 7) and Our Process (Step 8) with the client.
6. After the call: forward the `.txt` to Alex, drop the `.json` into Lexi.

## What's mocked vs. real

- ✓ Wizard captures everything end-to-end
- ✓ Auto-download both intake files on Confirm
- ✗ Auto-email to john@/alex@ — not wired (planned via Cloudflare Worker later)
- ✗ Auto-post to Notion — not wired (planned)

For tomorrow's call, manual delivery is fine: download → forward.

## Docs

- [Discovery Playbook](docs/DISCOVERY-PLAYBOOK.md) — how to run the call
- [Lexi Build Brief](docs/LEXI-BUILD-BRIEF.md) — intake JSON schema + ingestion notes
- [Internal Spec](docs/INTERNAL-SPEC.md) — the bigger product picture

## Tech

Single static HTML file. React via CDN, Tailwind via CDN, no build step. Served as a GitHub Pages site.

To run locally: just open `index.html` in a browser. Or `python3 -m http.server 5182` from this folder.

---

*Maintained by Kei 🌊 for Strategy Ninjas.*
