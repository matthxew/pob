# Portrait Analyzer

A single-page web app that uses Claude's vision model to score and critique portrait photographs against a 10-criterion rubric. Open the page, paste your Anthropic API key, drop in a portrait, and get back a 0–500 weighted score, per-criterion breakdown, written strengths and weaknesses, and forward-looking advice for your next shoot.

It's one HTML file. No build step, no server, no database, no tracking.

## How it works

The whole app is `index.html`. Open it in a browser and it runs.

- The rubric, scoring math, and UI live in the file itself.
- Your API key is stored only in your browser's `localStorage`.
- When you click "Analyze", the browser POSTs the image and rubric directly to `api.anthropic.com`. Nothing passes through any server the author controls.
- Past analyses are saved in `localStorage` so you can revisit them.

The "direct from browser" part uses Anthropic's `anthropic-dangerous-direct-browser-access` header. This is fine for a personal tool — the only person whose key gets sent is the person who pasted it.

## Setup

1. Get an Anthropic API key from [console.anthropic.com](https://console.anthropic.com). Add a small amount of credit (a few dollars goes a long way; see costs below).
2. Open `index.html` in your browser, or visit your hosted version.
3. Open the **Settings** tab and paste the key. Save.
4. Click **Test connection** to confirm it works.
5. Go to **Analyze**, drop in a portrait, click **Analyze with Claude**.

## Cost per analysis

Roughly, per portrait:

| Model              | Typical cost per analysis | Speed       |
| ------------------ | ------------------------- | ----------- |
| Claude Opus 4.7    | 5–25¢                     | 15–45s      |
| Claude Sonnet 4.6  | 1–5¢                      | 8–20s       |
| Claude Haiku 4.5   | under 1¢                  | 5–10s       |

Opus gives the best critique; Haiku is fine for a fast first pass. You can switch in Settings.

The system prompt is cached on Anthropic's side after the first call, so subsequent analyses in the same five-minute window pay ~10% on the cached portion.

## Hosting

The simplest options for a static one-file app:

**GitHub Pages (free):**

```sh
git init
git add index.html README.md
git commit -m "Initial commit"
gh repo create portrait-analyzer --public --source=. --push
```

Then in the repo on github.com: **Settings → Pages → Source: main → /(root) → Save**. Your site will be at `https://<your-username>.github.io/portrait-analyzer/`.

**Vercel (free hobby plan):**

Drag the folder into the Vercel dashboard, or run `vercel` in this directory. Done — no config needed.

**Or just open the file locally:**

`open index.html` works too. The localStorage persists per browser, so the experience is the same.

## The rubric

Each criterion is scored 0–5 and weighted. Weights sum to 100, so the total runs 0–500.

| #  | Criterion                  | Weight |
| -- | -------------------------- | ------ |
| 1  | Subject Presence           | 15     |
| 2  | Emotional Register         | 12     |
| 3  | Visual Craft               | 12     |
| 4  | Composition                | 10     |
| 5  | Authenticity               | 10     |
| 6  | Storytelling               | 10     |
| 7  | Setting & Context          | 8      |
| 8  | Originality                | 8      |
| 9  | Legibility at Small Size   | 8      |
| 10 | Mood & Atmosphere          | 7      |

Tiers:

- **430+** — Excellent. Award-tier work.
- **380–429** — Strong. Likely shortlist material.
- **300–379** — Solid. Real virtues, room to refine.
- **Below 300** — Rough draft. Use the feedback to plan a stronger version.

The full descriptors for each 0–5 rating are visible in the **Rubric** tab inside the app.

## Privacy

- No analytics, no third-party scripts, no telemetry.
- Your API key is stored in `localStorage`. Anyone with access to your browser can read it. Don't paste a key on a shared computer; clear it from Settings when you're done.
- Photos are sent to Anthropic. They are not stored by the author of this app — but they are processed by Anthropic per their [usage policies](https://www.anthropic.com/legal/usage-policy).
- If you fork this and host it publicly, the deployed URL is reachable by anyone who has it. Each visitor needs to paste their own API key — it doesn't share across visitors.

## Customizing

The rubric is defined as a JavaScript array near the top of the script section in `index.html`, marked with `// ---------- Rubric (10 criteria, weights sum to 100) ----------`. Each entry has:

- `id` — internal key, must match between rubric and AI response
- `name` — shown in UI
- `weight` — integer; weights across all entries should sum to 100
- `measures` — short description of what the criterion captures
- `descriptors` — exactly six strings, one per 0–5 rating

Add or remove criteria as needed. The score math will pick up the new weights automatically. Tier thresholds (`tierOf`) are absolute scores, so adjust them if you change the max possible score.

## Limits

- Image size is capped at ~12 MB. For best results, downsize the long edge to ~2000–2500 px before uploading — anything larger costs more in tokens without changing the analysis.
- `localStorage` is finite (typically ~5–10 MB per origin). The app keeps the most recent 50 analyses and trims older ones when storage is full.
- Switching browsers or devices means starting over — there is no cross-device sync. Use **History → Export JSON** to take your data with you.
- The model can be wrong. The score is a structured second opinion, not a verdict.

## License

MIT. Do whatever you want with it.
