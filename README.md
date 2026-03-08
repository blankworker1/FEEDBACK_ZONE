# FEEDBACK.ZONE

*A public listening project.*

[Link to Website](https://blankworker1.github.io/FEEDBACK_ZONE/)

---

FEEDBACK.ZONE is an anonymous voice art installation that turns the street into a confessional. Posters mounted across a city invite passersby to scan a QR code and speak to say something they might not say anywhere else. No name. No account. No record of who called.

What remains is only the aggregate: mood patterns, recurring words, emotional weather. A portrait of a place, rendered in the voices of strangers.

---

## The experience

A printed poster. A QR code. A phone to your ear.

When someone scans the code, a brief AI-guided conversation begins - warm, unhurried, genuinely curious. The AI asks a simple opening question and listens. After a short exchange, the call ends. The voice disappears.

Across the city, a live dashboard accumulates what was felt, what was mentioned, what was on people's minds without ever knowing who said it.

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
     index.html        the call interface (served via QR code)
     dashboard.html    the public live dashboard (share this URL publicly)
     poster.html       print-ready A3 poster template with live editor
     worker.js         serverless backend (deploy to Cloudflare Workers)
     schema.sql        database schema (run once in Supabase)
     README.md         this file
```

Once deployed, the two public URLs will be:
- Call interface: `https://yourusername.github.io/FEEDBACK_ZONE/`
- Live dashboard: `https://yourusername.github.io/FEEDBACK_ZONE/dashboard.html`

---

## Infrastructure overview

```
Poster QR code
     |
Poster Editor
     |
index.html (GitHub Pages)
     |
ElevenLabs Conversational AI   voice conversation happens here
     | webhook (call ended)
Cloudflare Worker              extracts sentiment + themes via Claude API
     |
Supabase                       stores anonymised row only
     |
dashboard.html (GitHub Pages)  public, auto-refreshes every 30 seconds
```

All components either run on free tiers or cost a few cents per call at scale.

---

## Setup

### Prerequisites

Accounts needed all have free tiers:
- [GitHub](https://github.com)  static hosting via GitHub Pages
- [ElevenLabs](https://elevenlabs.io) conversational AI voice
- [Cloudflare](https://cloudflare.com) serverless worker
- [Supabase](https://supabase.com)  database
- [Anthropic](https://console.anthropic.com)  Claude API for analysis

---

### 1. Database Supabase

1. Create a new Supabase project
2. Open the SQL Editor and run the contents of `schema.sql`
3. Go to **Project Settings â†’ API** and copy:
   - Project URL
   - `anon` public key
   - `service_role` secret key (for the worker only keep this private)

---

### 2. Voice AI - ElevenLabs

1. Go to **Conversational AI Create Agent**
3. Set a system prompt that establishes the tone. Example:

   > *You are a warm, unhurried listener called FEEDBACK.ZONE. Greet the caller simply and ask how they're feeling today. Listen to their response, reply with genuine curiosity â€” one or two exchanges at most. Then close the call gently with a thank you. Keep the whole conversation under two minutes.*

4. Under **Security**, set Allowed Domains to your GitHub Pages URL
5. Under **Webhooks**, add: `https://your-worker.workers.dev/webhook/elevenlabs`
6. Add a conversation variable: name `poster_id`, mapped to URL parameter `poster`
7. Copy your **Agent ID**

---

### 3. Analysis API - Anthropic

The Cloudflare Worker uses Claude Haiku to extract sentiment and themes from each call transcript. You need an API key.

1. Create an account at [console.anthropic.com](https://console.anthropic.com)
2. Go to **API Keys Create Key** and copy the key somewhere safe it's only shown once
3. Add a small amount of credit (a few dollars covers thousands of calls at Haiku pricing)

You'll add this key to the worker as a secret in the next step. Do not put it in any file that gets committed to GitHub.

---

### 4. Backend worker - Cloudflare

```bash
npm install -g wrangler
wrangler login
wrangler init feedback-zone-worker
# copy worker.js into the project
```

Before deploying, create a `wrangler.toml` file in the worker project folder:

```toml
name = "feedback-zone-worker"
main = "worker.js"
compatibility_date = "2024-01-01"
```

Then add your four secrets â€” these are stored securely by Cloudflare and never exposed in code:

```bash
wrangler secret put ANTHROPIC_API_KEY       # from step 3
wrangler secret put SUPABASE_URL            # from step 1
wrangler secret put SUPABASE_SERVICE_KEY    # from step 1
wrangler secret put WEBHOOK_SECRET          # choose any random string; you'll set the same value in ElevenLabs
wrangler deploy
```

The worker URL will be `https://feedback-zone-worker.your-subdomain.workers.dev`.

---

### 5. Configure the frontend files

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

### 6. Deploy to GitHub Pages

1. Push all six files to a GitHub repository
2. Go to **Settings > Pages > Source > Deploy from branch > main > / (root)**
3. Your app will be live at `https://yourusername.github.io/your-repo-name`

---

### 7. Print the posters

`poster.html` is a print-ready A3 template with a live editor built in. Open it in any browser to customise, add your QR code, and download.

**Editing for each location:**

Open `poster.html` in a browser. Use the editor panel at the top to update:
- **Headline** the large display text
- **Provocation** the line that stops someone in the street
- **Call to action**  the instruction beneath the QR code
- **Location tag**  appears top-right (e.g. *North Quarter*)

Changes update the poster preview in real time.

**Adding the QR code:**

Each poster needs its own QR code pointing to a unique URL with a location identifier:

```
https://yourusername.github.io/feedback-zone/?poster=city-centre
https://yourusername.github.io/feedback-zone/?poster=north-quarter
https://yourusername.github.io/feedback-zone/?poster=riverside
```

Generate a QR code for each URL using any free QR generator (qrcode.me, qr-code-generator.com). Then click the QR box directly on the poster preview â€” a file picker will open. Select your saved QR image and it will appear on the poster immediately. Click the box again at any time to swap it out.

**Downloading:**

Click ** Download poster** to save the poster as a PNG file. The filename is generated automatically from the location tag (e.g. `feedbackzone-north-quarter.png`). Send this file to a printer or print shop. Recommended: A3, black ink only, on white stock. Scales to A4 if needed.

The `poster` parameter in each QR code URL flows automatically through the entire pipeline and appears as a location filter on the dashboard.

---

### 8. Test end-to-end

1. Scan a QR code on a mobile device
2. Complete the privacy screen, grant microphone access, have a conversation
3. Check your Supabase table a new anonymised row should appear within seconds of the call ending
4. Open `dashboard.html`  data appears on the next 30-second refresh

---

## Browser compatibility

The app uses the browser microphone API, which behaves differently across mobile browsers.

| Browser | Status | Notes |
|---|---|---|
| iOS Safari 15+ | Supported | Requires a user tap before audio plays the app handles this |
| Android Chrome 90+ | Supported | Works as expected |
| Firefox Mobile 90+ | Supported | Works as expected |
| iOS Chrome / Firefox | Limited | These browsers use Safari's engine on iOS mic access may vary |
| In-app browsers | Avoid | Facebook, Instagram, LinkedIn browsers often block mic access |

**For posters:** consider adding a small line of copy such as *"Open in Safari or Chrome"* if your audience is likely to scan from within a social media app.

---

## ElevenLabs: setting up the poster_id variable

This step is the most easily missed in the setup. Without it, all calls will arrive in Supabase with a null `poster_id` and location filtering on the dashboard won't work.

In your ElevenLabs agent settings:

1. Go to **Agent Variables**
2. Add a new variable: name it exactly `poster_id`
3. Set the source to **URL parameter**
4. Set the parameter name to `poster`

ElevenLabs will now read `?poster=north-quarter` from the URL when a call starts and pass it through to the webhook payload automatically. No code changes needed.

---

## Troubleshooting

**Microphone not working on iOS**
Safari on iOS requires a user gesture before accessing the microphone. The app includes a tap-to-start button for this reason. If the mic still fails, check Settings â†’ Safari â†’ Microphone â†’ and ensure the site is set to Allow.

**Webhook not firing after a call**
Check that the webhook URL in ElevenLabs matches your deployed worker URL exactly, including `https://` and the `/webhook/elevenlabs` path. Verify the `WEBHOOK_SECRET` value set in Cloudflare matches the secret header configured in ElevenLabs.

**Supabase rows not appearing**
The most likely cause is a Row Level Security policy blocking the write. Open the Supabase dashboard, go to **Table Editor calls**, and check if rows are appearing there directly. If they are, the issue is in the dashboard's read query. If they aren't, check the worker logs in the Cloudflare dashboard for error messages.

**Dashboard showing no data**
Confirm `YOUR_SUPABASE_URL` and `YOUR_SUPABASE_ANON_KEY` are correctly set in `dashboard.html`. The anon key is the public-facing one (not the service key). Also check that the Supabase RLS policy allows public reads for the last 24 hours as defined in `schema.sql`.

**Worker deployment failing**
Ensure `wrangler.toml` exists in the worker project folder with the correct `name` and `compatibility_date` fields before running `wrangler deploy`.

---

## Privacy and legal considerations

FEEDBACK.ZONE is designed to be privacy-preserving by architecture â€” no voice, no transcript, and no personal data is stored. However, if you deploy this in a public space in the EU or UK, a few points are worth considering:

- **GDPR**: Even anonymised aggregate data may require a brief privacy notice if it could theoretically be linked back to an individual. The privacy screen in the app addresses this, but you may want legal review for large-scale deployments.
- **Public space recording laws**: The app does not record audio, but scanning a QR code and speaking into a phone in a public space is inherently visible. No additional legal obligations apply beyond normal conduct.
- **Data retention**: The schema includes an optional `pg_cron` job to auto-delete rows older than 30 days. Consider enabling this for any deployment that runs beyond a short-term installation.

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

- **Real-time scrolling feed**  anonymised snippets surfacing live on the dashboard
- **Map visualisation** sentiment and theme distribution plotted by poster location across the city
- **Multi-day archive view** trends across days or weeks
- **Embeddable widget** dashboard panels usable in external sites or exhibition contexts

---

## License

This project is released as open source. Do with it what feels right. If you deploy it somewhere, consider leaving a note about what you found.




 
