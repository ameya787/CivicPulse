# CivicPulse

**Many voices. One clear priority.**
CivicPulse is a small civic issue-reporting web app I built for the Genspark hackathon. 
It tries to fix a simple problem: when a pothole or a garbage pile is reported by ten different people, 
the city sees ten separate complaints instead of one issue.

## What it does
Citizens describe a problem, drop a pin on the map and optionally add a photo. The app guesses the category, checks whether 
similar reports already exist nearby, and either adds the report to an existing issue or starts a new one. Every issue gets 
a priority score from 0 to 100.
City authorities get a ranked list of issues, a map and analytics. Each score comes with a breakdown, so you can see why 
an issue ranks where it does.
The demo uses a fictional city called **Riverton**, with 44 seeded reports across 8 neighborhoods that group into about 
9-10 issues.

## Features
**For citizens**
- 3-step report wizard: describe, locate, review and submit
- Live category suggestion with confidence, highlighted keywords and urgency flags while you type
- Photo upload with client-side resizing (the photo never leaves the browser)
- Map pin selection, "use my location", and a notice when similar open issues are nearby
- My Reports with a status tracker (Submitted, Under Review, Assigned, In Progress, Resolved)
- Community feed and map with upvotes that change priority

**For authorities**
- Command dashboard with KPI cards, a ranked priority queue and a live map
- Map layers: clusters, individual reports and a heatmap
- Filters, search and sorting on the queue; assign and status actions
- Cluster detail page with a summary, a "Why this priority?" breakdown, an escalation gauge, report timeline, notes and status history
- Analytics page with charts, a grouping efficiency card and CSV export
- Print-friendly summary

**General**
- Citizen / Authority role switch (no login)
- Light and dark mode, reduced-motion setting
- Responsive layout down to 360 px
- Guided **Demo Mode** of about two minutes

## How grouping works
When a new report comes in, it is compared with every open cluster of the same (or a closely related) category. It joins a cluster if any of these is true:
- it is within 150 m of the cluster centre and the text similarity is at least 0.30
- it is within 60 m, whatever the wording
- it is within 300 m and the text similarity is at least 0.55

If several clusters match, the one with the best score wins: `0.6 * (1 - distance/300) + 0.4 * similarity`. If none match, a new cluster is created. Resolved clusters are not joined.
Text similarity is half Jaccard overlap and half TF-IDF cosine, after stop-word removal, light stemming and a small synonym map. Distance uses the Haversine formula.

## How the priority score works
The score is a weighted sum of five factors, each scaled 0-100:
| Factor | Weight | Based on |
|---|---|---|
| Severity | 30% | Base severity of the category, plus 5 per urgency keyword |
| Volume | 20% | Number of reports and upvotes |
| Safety and sensitive location | 20% | Near a school or hospital (highest), transit hub, market, or none |
| Age | 15% | Days the issue has been open (zero once resolved) |
| Urgency language | 15% | Words like "dangerous", "accident", "child", "exposed" |
Levels: **Critical** 75+, **High** 55-74, **Medium** 35-54, **Low** below 35. Resolved issues always show as Low. The full breakdown is visible on each cluster page.

## What is real and what is simulated
Parts of the AI are real computation and parts are there for the demo. The app labels each one with a **Live logic** or **Simulated** badge.
| Real (computed in the browser) | Simulated (demo only) |
|---|---|
| Category classifier (weighted keywords and phrases) | Timing and typing effect of the "AI analysis" steps |
| Text similarity | Photo analysis (no computer vision) |
| Distance calculation | Cluster summaries (templates filled with real numbers) |
| Issue grouping | Department routing and resolution time estimates |
| Priority scoring | Escalation risk gauge |
| Urgency detection | |
| Analytics | |
There is an optional "live LLM" setting in the settings drawer. It is off by default and the app does not depend on it. 

## Run it locally
No install or build step is needed.
1. Clone the repo:
```bash
git clone https://github.com/ameya787/CivicPulse.git
cd CivicPulse
```
2. Open `index.html` in a browser.
Or serve it locally:
```bash
python -m http.server 8000
```
Then visit `http://localhost:8000`.
Leaflet and the Inter font load from CDNs. If they are unavailable, the app falls back to a built-in schematic map and system fonts.

## Try the demo
Click **Start Demo** in the top bar. The app resets to the seeded data and walks through the flow in about two minutes:
1. Landing page and live stats
2. A citizen reports a dangerous pothole on Lakeview Road
3. The live analysis groups it with the existing Lakeview pothole cluster and raises it to Critical
4. Switch to the authority dashboard, where that cluster is now ranked first
5. Open the cluster to see the priority explanation, then assign it
6. Analytics: grouping efficiency and hotspots
Controls: Space to pause or play, arrow keys for next and back, Esc to exit.
To try it by hand, open Settings and use "Load the prepared demo report", submit it, then switch to Authority.

## Deploy to GitHub Pages
1. In the repo go to **Settings > Pages**.
2. Under **Build and deployment**, set the source to **Deploy from a branch**, choose the `main` branch and the `/ (root)` folder, and save.
3. After a minute or two the site is live at https://ameya787.github.io/CivicPulse/.

## Project structure
```
CivicPulse/
├── index.html        # the whole app: styles, scripts and markup
├── README.md
├── LICENSE
└── docs/
    └── screenshots/  # add dashboard.png and others here
```

Inside the script, the code is organised in sections:
| Section | Purpose |
|---|---|
| `core` | Constants, seeded random numbers, date and text helpers |
| `ai-real` | Classifier, similarity, grouping, priority, analytics |
| `ai-sim` | Routing tables, photo analysis, summaries, escalation risk |
| `seed` | Riverton neighborhoods and the 44 demo reports |
| `ui` | Shared components: charts, map wrapper, badges, modals |
| `pages` | Landing, report wizard, my reports, community, dashboard, cluster detail, analytics, about |
| `demo` | Guided Demo Mode |
| `app` | State, localStorage persistence, router, settings, boot |



## Tech
Plain HTML, CSS and JavaScript, with [Leaflet](https://leafletjs.com/) for maps. Charts are drawn in the app itself. 
All data is stored in the browser's `localStorage` under the key `civicpulse:v1`. There is no backend and no account 
system.

## Limitations
- All data is fictional and lives only in your browser. Clearing site data or using "Reset demo data" brings back the seed.
- The Citizen / Authority switch is a demo toggle, not real access control.
- Photo analysis, summaries, routing and the escalation gauge are simulated.
- The classifier is keyword-based, so unusual wording can land in "Other".
- Map tiles need an internet connection; the fallback map shows pins without a street layer.

## Ideas for next
- Real image analysis for photos
- Multilingual reporting
- Connecting to a real ticketing or municipal system
- SMS or WhatsApp reporting
- Adjustable priority weights for authorities
- Manual fix for wrongly grouped reports

## Built with
Built with [Genspark](https://www.genspark.ai/), with AI assistance for planning, code and writing.

## Author
Ameya - [GitHub](https://github.com/ameya787) 
