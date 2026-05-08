# RateWolf — Hotel Navis · Opatija

Competitive rate intelligence dashboard. Tracks Navis vs. 7 luxury competitors across Booking.com daily, with an editorial-luxury single-page UI.

Built for **Revenue Wolves** by David Atlija.

---

## What it does

- Daily snapshot of Navis + 7 comp set hotels on Booking.com (rates, room types, refundable flags, sold-out signals)
- Two scan strategies per hotel:
  - **Midweek** — lowest rate found across Sun/Mon/Tue/Wed/Thu, week by week
  - **Weekend** — explicit Fri & Sat rates for the next 4 weekends
- Horizon: current month + 3 months forward
- Dashboard surfaces: KPI strip, full rate matrix (heatmap with delta colouring), trend line (Navis vs market median), anomaly feed of notable moves since previous snapshot

---

## Architecture

```
ratewolf-navis/
├── index.html                          # Single-file dashboard (PIN-protected)
├── config/
│   └── comp_set.json                   # The 8 hotels + scan parameters
├── data/
│   └── prices.json                     # Committed snapshot history
├── instructions/
│   └── claude-chrome-workflow.md       # Brief the Claude in Chrome agent reads
└── README.md
```

**Data flow:**

1. Open Chrome, open Claude in Chrome side panel
2. Hand it `instructions/claude-chrome-workflow.md` and tell it to run
3. Agent navigates Booking.com for each hotel × each target date, returns one JSON
4. You open the dashboard, click `+ New Snapshot`, paste JSON → it's saved to localStorage and renders
5. When you want to commit it permanently: click `⤓ Export`, replace `data/prices.json` in the repo, push, Netlify redeploys

The dashboard merges committed history (`data/prices.json`) with any local-only snapshots (localStorage) so you can keep working between commits.

---

## First-time setup

### 1. Create the GitHub repo

```bash
cd ratewolf-navis
git init
git add .
git commit -m "Initial commit: RateWolf MVP for Hotel Navis"
gh repo create ratewolf-navis --private --source=. --push
```

(Or use the GitHub web UI — drag the folder into a new private repo.)

### 2. Deploy to Netlify

- Netlify → Add new site → Import from Git → select the repo
- Build command: *(none)*
- Publish directory: `/` (root)
- Deploy

You'll get a URL like `ratewolf-navis-xyz.netlify.app`. Add a custom domain later if useful (`ratewolf.revenuewolves.com` or similar).

### 3. Set the PIN

Default PIN is `1995`. To change it:

1. Open the deployed dashboard
2. Open browser console (Cmd+Option+J on Mac)
3. Paste:
   ```js
   localStorage.setItem('rw_pin_v1', String('YOUR_NEW_PIN'.split('').reduce((h,c)=>((h<<5)-h)+c.charCodeAt(0)|0,0)));
   ```
   Replace `YOUR_NEW_PIN` with a 4-digit string.
4. Reload, enter the new PIN.

(Phase 2 would move auth to a Netlify Function — for MVP, this is fine since the dashboard is private and the data isn't sensitive.)

### 4. Verify Booking URLs

The URLs in `config/comp_set.json` are best guesses based on Booking's slug conventions. On the **first agent run**, the workflow tells Claude to verify each URL points to the right hotel and report corrections in `session_notes`. Update the config with the correct slugs and commit.

---

## Daily ritual (5 minutes)

1. **Open Chrome** with Claude in Chrome extension active
2. **Tell Claude**: *"Read `instructions/claude-chrome-workflow.md` from this repo and run RateWolf for today."*
   - Or paste the workflow content directly into the chat
3. **Wait ~10–15 minutes** while it scans (faster after the first run when it knows the layout)
4. **Copy the JSON** Claude prints at the end
5. **Open the dashboard**, enter PIN, click `+ New Snapshot`, paste, ingest
6. Review the anomaly feed — that's where the value lives day-to-day

**Weekly:** Click Export, replace `data/prices.json`, commit, push. This bakes the week's snapshots into the repo so the dashboard has them on a fresh device.

---

## Phase 2 ideas (don't build yet)

- **Make.com automation**: scheduled trigger → Browserless headless Chrome → JSON to Make → POST to a Netlify Function → Netlify Blobs → dashboard reads Blobs. Removes the manual paste step.
- **Multi-property**: parameterise the dashboard to handle several Revenue Wolves clients (`?client=navis`, `?client=ikador-resort`, etc.) reading separate config + data files.
- **Email digest**: Friday afternoon Make scenario reads latest snapshots, calls Claude API to generate a 1-page commentary, emails the GM.
- **Price calendar**: monthly view (one cell per day per hotel) for forward-looking pacing analysis.
- **Booking.com partner API**: if Navis gives Revenue Wolves access, replace public scraping with the Channel Manager API for ground-truth numbers (and add internal data points like LOS, ADR by segment).

---

## Notes on Booking.com

- Booking is adversarial against scrapers but Claude in Chrome runs in a real browser with a real user profile. So far this works. If they roll out tighter detection, switching to direct hotel-page URLs (which is what this workflow does) is more resilient than search-result scraping.
- Always browse in a fresh state (no Genius login) so the agent captures the **public** rate, not member rates.
- If you start seeing CAPTCHAs, add a 2–3 second random delay between hotels in the workflow. Don't run the agent more than once a day.

---

## Branding

The dashboard footer says *design by David Atlija · Revenue Wolves*. The brand mark "RateWolf" is intentional — slot it into the wider Revenue Wolves product family if/when you add a public site.

---

**v1.0** · MVP · May 2026
