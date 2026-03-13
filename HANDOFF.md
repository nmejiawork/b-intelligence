# HANDOFF — B Intelligence
*Last updated: 2026-03-12*
*Deploy: https://b-intelligence.vercel.app*

## What This App Does
B Intelligence is a single-page YouTube analytics and content creation tool built specifically for the Humble Conviction (HC) channel. Brian uses it to monitor channel performance, generate AI-assisted titles and descriptions, analyze thumbnail effectiveness, and surface trending topics — all from one screen without leaving to other tools.

## Tech Stack
- Frontend: React 18 (UMD via CDN) + JSX transpiled at runtime by Babel Standalone
- Styling: Tailwind CSS (CDN) + inline styles
- Charts: Chart.js (CDN)
- Backend/Data: No server — YouTube Data API v3 (client-side) + Claude AI API (client-side)
- Hosting: Vercel (static file deployment via `@vercel/static`)
- Repo: https://github.com/brianhecht/b-intelligence

## Folder Structure
```
b-intelligence/
├── index.html        ← entire app — all React components, logic, styles (1948 lines)
├── vercel.json       ← static deploy config, routes all traffic to index.html
├── HANDOFF.md        ← this file
└── .gitignore
```

There is no `package.json`, no `node_modules`, no build step. Everything runs from a single HTML file via CDN scripts. This is intentional — zero-dependency, zero-maintenance deployment.

## Key Architecture Notes

**Runtime transpilation:** JSX is compiled in the browser by Babel Standalone (`<script type="text/babel">`). No build step is needed, but the browser does a 1–2 second compile on first load.

**HC_CONTEXT (lines 452–562):** Hardcoded intelligence object containing HC-specific topics, voice guidelines, title formulas (8 VidIQ patterns), and performance insights. This is the "brain" behind all template generation. Update this when strategy changes.

**localStorage keys:**
- `bi_yt_key` — YouTube API key (falls back to hardcoded default)
- `bi_yt_channel` — YouTube channel ID (falls back to hardcoded default)
- `bi_claude_key` — Claude API key (no default — user must enter manually)

**Default credentials (line 577):**
```js
this.key = localStorage?.getItem("bi_yt_key") || "AIzaSyDtc1ZZBTOFzEy5aqFy4fKIZkqTOJkvKs4";
this.channelId = localStorage?.getItem("bi_yt_channel") || "UCqAhVRJlLyY86vWAE5s_xhA";
```
The YouTube key is stored in GCP Console under project `b-intelligence`. When it expires, replace the string on line 577 and commit.

## The 6 Modules

| Module | What it does |
|--------|-------------|
| **Dashboard** | Channel stats (subscribers, views, videos) + HC Pattern Intelligence + recent video performance table |
| **Trend Radar** | Hardcoded HC niche trends + live YouTube search for trending topics |
| **Title Generator** | 8 VidIQ template titles based on topic input + optional Claude AI title variants |
| **Description Builder** | VidIQ headline-first description template + optional Claude AI description |
| **Thumbnail Reviewer** | Drag-and-drop image upload + automated analysis (aspect ratio, text density, contrast, composition) |
| **Settings** | YouTube API key/channel config + Claude API key input + connection status indicators |

## Current Status
Fully functional and deployed. All 6 modules work. YouTube connection is live with the refreshed API key. Claude integration is optional and requires the user to provide their own API key in Settings.

## What Changed This Session
- Confirmed the YouTube API key update (`AIzaSyAwr9WgpteOeFHuX-Lez3ZfWVumXI25u0Y` → `AIzaSyDtc1ZZBTOFzEy5aqFy4fKIZkqTOJkvKs4`) was already committed and deployed from the prior session (commit `09d4a3a`). No code changes were needed this session — the app was already live with the new key.
- Verified production deployment at b-intelligence.vercel.app shows "YouTube Connected" status.
- Wrote this HANDOFF.md and a separate User Guide.

## Known Issues / Bugs
- **Thumbnail reviewer is client-side only** — analysis runs on basic heuristics (aspect ratio, estimated text coverage, brightness variance). It does not use any ML/CV model. Results are rule-based, not perception-based.
- **Trend Radar niche trends are hardcoded** — the "HC Niche Trends" section in Trend Radar uses static data from `HC_CONTEXT`. Only the YouTube search results are live. Trends won't age out automatically.
- **Claude titles/descriptions require API key** — if Brian hasn't entered a Claude key in Settings, the AI generation buttons show an error. Template-based generation always works without Claude.
- **No error UI for quota exhaustion** — if the YouTube API key hits its daily quota limit (10,000 units/day on free tier), the app will show blank data with no helpful message.

## Backlog / Next Steps
- Add real-time trend data source to replace hardcoded HC_CONTEXT trends
- Build a "Publish Queue" or content calendar view
- Add Claude vision for thumbnail review (replace heuristic-only analysis)
- Surface quota usage warning when YouTube API is near limit
- Consider migrating to a proper Next.js build if the app grows beyond ~2500 lines

## Design Decisions
**Single HTML file:** Chosen for zero-friction deployment and maintenance. No build pipeline to break, no dependency rot, no CI/CD complexity. The tradeoff is that the file will get unwieldy past ~3000 lines — at that point, consider a proper Next.js migration.

**Hardcoded HC_CONTEXT:** All channel intelligence (topics, formulas, voice) lives in the file rather than a database. This makes it portable and fast, but requires a code edit + deploy to update strategy. When the strategy changes enough that updates feel annoying, move HC_CONTEXT to Firestore.

**Client-side API calls:** Both YouTube and Claude APIs are called directly from the browser. This avoids needing a backend, but means the API key is technically visible in localStorage. Fine for a single-user internal tool — not appropriate if this ever becomes multi-user or public.

**No auth:** App is open to anyone with the URL. Intentional — this is a personal tool for Brian, not a product.

## Environment Variables
No `.env.local` or Vercel environment variables are required. API keys are hardcoded as defaults in `index.html` (YouTube) or entered by the user at runtime in Settings (Claude). 

If you ever want to move the YouTube key out of source code, add it as a Vercel env var and update the fetch call to hit a Vercel serverless function instead. Not worth it for a single-user tool.

## QA Checklist
- [ ] Dashboard loads channel stats (subscribers, total views, video count)
- [ ] Dashboard shows recent videos with view counts
- [ ] Trend Radar YouTube search returns results for a test query
- [ ] Title Generator produces 8 template titles for a topic input
- [ ] Title Generator Claude button works when Claude key is set in Settings
- [ ] Description Builder produces a formatted description from title + takeaways
- [ ] Thumbnail Reviewer accepts a drag-dropped image and shows analysis
- [ ] Settings page shows "YouTube Connected" with green indicator
- [ ] Settings page saves a custom YouTube key to localStorage
- [ ] Mobile layout is readable (sidebar collapses gracefully)

## Open Questions
- Should HC_CONTEXT be updated to reflect any recent changes to Brian's content strategy or title formulas?
- Is the current YouTube API quota (10K units/day) sufficient for daily use, or should we request a quota increase?
