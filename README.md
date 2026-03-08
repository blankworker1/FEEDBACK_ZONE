# FEEDBACK.ZONE

*A public listening project.*

[Link to Website](https://blankworker1.github.io/FEEDBACK_ZONE/)

---

FEEDBACK.ZONE is an anonymous voice art installation that turns the street into a confessional. Posters mounted across a city invite passersby to scan a QR code and speak — to say something they might not say anywhere else. No name. No account. No record of who called.

What remains is only the aggregate: mood patterns, recurring words, emotional weather. A portrait of a place, rendered in the voices of strangers.

---

## The experience

A printed poster. A QR code. A phone to your ear.

When someone scans the code, a brief AI-guided conversation begins — warm, unhurried, genuinely curious. The AI asks a simple opening question and listens. After a short exchange, the call ends. The voice disappears.

Across the city, a live dashboard accumulates what was felt, what was mentioned, what was on people's minds — without ever knowing who said it.

---

## What is and isn't collected

**Collected (anonymised):**
- Sentiment score (a number, not a transcript)
- General theme categories (e.g. *Wellbeing*, *Community*, *Work*)
- Frequently mentioned words
- Call duration
- Which poster location initiated the call

**Never collected:**
- Voice recordings
- Transcripts
- Names, locations, or any identifying information
- IP addresses or device data

The pipeline is designed so that raw speech never touches a database. An AI model processes the conversation in real-time and discards it, writing only the extracted signal.

---

## Repository structure

```
feedback-zone/
├── index.html       — the call interface (served via QR code)
├── dashboard.html   — the public live dashboard
├── poster.html      — print-ready A3 poster template with live editor
├── worker.js        — serverless backend (deploy to Cloudflare Workers)
├── schema.sql       — database schema (run once in Supabase)
└── README.md        — this file
```

---

## Infrastructure overview

```
Poster QR code
     ↓
index.html (GitHub Pages)
     ↓
ElevenLabs Conversational AI  ← voice conversation happens here
     ↓ webhook (call ended)
Cloudflare Worker             ← extracts sentiment + themes via Claude API
     ↓
Supabase                      ← stores anonymised row only
     ↓
dashboard.html (GitHub Pages) ← public, auto-refreshes every 30 seconds
```

All components either run on free tiers or cost a few cents per call at scale.

---

## Setup

### Prerequisites

Accounts needed — all have free tiers:
- [GitHub](https://github.com) — static hosting via GitHub Pages
- [ElevenLabs](https://elevenlabs.io) — conversational AI voice
- [Cloudflare](https://cloudflare.com) — serverless worker
- [Supabase](https://supabase.com) — database
- [Anthropic](https://console.anthropic.com) — Claude API for analysis

---

### 1. Database — Supabase

1. Create a new Supabase project
2. Open the SQL Editor and run the contents of `schema.sql`
3. Go to **Project Settings → API** and copy:
   - Project URL
   - `anon` public key
   - `service_role` secret key (for the worker only — keep this private)

---

### 2. Voice AI — ElevenLabs

1. Go to **Conversational AI → Create Agent**
2. Set a system prompt that establishes the tone. Example:

   > *You are a warm, unhurried listener called FEEDBACK.ZONE. Greet the caller simply and ask how they're feeling today. Listen to their response, reply with genuine curiosity — one or two exchanges at most. Then close the call gently with a thank you. Keep the whole conversation under two minutes.*

3. Under **Security**, set Allowed Domains to your GitHub Pages URL
4. Under **Webhooks**, add: `https://your-worker.workers.dev/webhook/elevenlabs`
5. Add a conversation variable: name `poster_id`, mapped to URL parameter `poster`
6. Copy your **Agent ID**

---

### 3. Backend worker — Cloudflare

```bash
npm install -g wrangler
wrangler login
wrangler init feedback-zone-worker
# copy worker.js into the project
wrangler secret put ANTHROPIC_API_KEY
wrangler secret put SUPABASE_URL
wrangler secret put SUPABASE_SERVICE_KEY
wrangler secret put WEBHOOK_SECRET
wrangler deploy
```

The worker URL will be `https://feedback-zone-worker.your-subdomain.workers.dev`.

---

### 4. Configure the frontend files

In `index.html`, replace:
```
YOUR_AGENT_ID_HERE
```
with your ElevenLabs Agent ID.

In `dashboard.html`, replace:
```
YOUR_SUPABASE_URL
YOUR_SUPABASE_ANON_KEY
```
with the values from step 1.

---

### 5. Deploy to GitHub Pages

1. Push all six files to a GitHub repository
2. Go to **Settings → Pages → Source → Deploy from branch → main → / (root)**
3. Your app will be live at `https://blankworker1.github.io/FEEDBACK_ZONE/`

---

### 6. Print the posters

`poster.html` is a print-ready A3 template with a live editor built in. Open it in any browser to customise and print.

**Editing for each location:**

Open `poster.html` in a browser. Use the editor panel at the top to update:
- **Headline** — the large display text
- **Provocation** — the line that stops someone in the street
- **Call to action** — the instruction beneath the QR code
- **Location tag** — appears top-right (e.g. *North Quarter*)
- **URL** — shown beneath the QR box

Changes update the poster preview in real time.

**Adding the QR code:**

Each poster needs its own QR code pointing to a unique URL with a location identifier:

```
https://yourusername.github.io/feedback-zone/?poster=city-centre
https://yourusername.github.io/feedback-zone/?poster=north-quarter
https://yourusername.github.io/feedback-zone/?poster=riverside
```

Generate a QR code for each URL using any free QR generator (qrcode.me, qr-code-generator.com). Then open `poster.html` in a text editor and find this comment:

```html
<!-- Replace this div with: <img src="your-qr-code.png" width="130" height="130" /> -->
```

Replace the placeholder div with an `<img>` tag pointing to your saved QR code image. Save, reopen in browser, and print.

**Print settings:**

File → Print → A3, black ink only, fit to page, no headers or footers. Scales to A4 if needed. The `poster` parameter flows automatically through the entire pipeline and appears as a location filter on the dashboard.

---

### 7. Test end-to-end

1. Scan a QR code on a mobile device
2. Complete the privacy screen, grant microphone access, have a conversation
3. Check your Supabase table — a new anonymised row should appear within seconds of the call ending
4. Open `dashboard.html` — data appears on the next 30-second refresh

---

## Running costs

| Service | Free tier | ~100 calls/day |
|---|---|---|
| GitHub Pages | Unlimited | $0 |
| Supabase | 50,000 rows | $0 |
| Cloudflare Workers | 100k requests/day | $0 |
| ElevenLabs | 250 min/month | ~$0.10/min after |
| Anthropic Claude Haiku | Pay as you go | ~$0.001/call |

At modest scale, the only meaningful cost is ElevenLabs call time.

---

## Extending the project

Features deliberately held back for future versions:

- **Real-time scrolling feed** — anonymised snippets surfacing live on the dashboard
- **Map visualisation** — sentiment and theme distribution plotted by poster location across the city
- **Multi-day archive view** — trends across days or weeks
- **Embeddable widget** — dashboard panels usable in external sites or exhibition contexts

---

## License

This project is released as open source. Do with it what feels right. If you deploy it somewhere, consider leaving a note about what you found.
