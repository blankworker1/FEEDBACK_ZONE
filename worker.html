/**
 * FEEDBACK.ZONE — Cloudflare Worker
 *
 * Flow:
 *   ElevenLabs webhook (call ended) →
 *   Extract poster_id from call metadata →
 *   Claude API → sentiment score + keywords + themes →
 *   Supabase insert (anonymised row only — no transcript stored)
 *
 * Deploy:
 *   1. npm install -g wrangler
 *   2. wrangler secret put ANTHROPIC_API_KEY
 *      wrangler secret put SUPABASE_URL
 *      wrangler secret put SUPABASE_SERVICE_KEY
 *      wrangler secret put WEBHOOK_SECRET     (match value in ElevenLabs dashboard)
 *   3. wrangler deploy
 *
 * Poster ID:
 *   Each poster's QR code points to a unique URL, e.g.:
 *     https://yoursite.github.io/?poster=north-quarter
 *   ElevenLabs passes this through in the call metadata.
 *   Configure your ElevenLabs agent to collect `poster_id`
 *   as a custom variable from the URL parameter.
 */

export default {
  async fetch(request, env) {
    if (request.method === 'OPTIONS') return new Response(null, { headers: corsHeaders() });

    const url = new URL(request.url);

    if (url.pathname === '/health')
      return json({ status: 'ok', ts: new Date().toISOString() });

    if (url.pathname === '/webhook/elevenlabs' && request.method === 'POST')
      return handleWebhook(request, env);

    return new Response('Not found', { status: 404 });
  }
};

// ════════════════════════════════════════════════════════
// WEBHOOK HANDLER
// ════════════════════════════════════════════════════════
async function handleWebhook(request, env) {
  try {
    const body = await request.json();

    // Verify shared secret
    const secret = request.headers.get('x-webhook-secret');
    if (env.WEBHOOK_SECRET && secret !== env.WEBHOOK_SECRET)
      return new Response('Unauthorised', { status: 401 });

    // Extract fields from ElevenLabs payload
    // Adjust these paths if ElevenLabs changes their webhook schema
    const transcript    = body?.transcript
                       || body?.conversation?.transcript
                       || '';
    const durationSecs  = body?.duration_seconds
                       || body?.conversation?.metadata?.duration
                       || 0;

    // poster_id: ElevenLabs can pass URL params as conversation variables
    // Set this up in your ElevenLabs agent as a collected variable named "poster_id"
    const posterId = body?.conversation?.variables?.poster_id
                  || body?.metadata?.poster_id
                  || null;

    if (!transcript)
      return json({ status: 'skipped', reason: 'no transcript' });

    const analysis = await analyseTranscript(transcript, env);

    await writeToSupabase({
      ...analysis,
      duration_seconds: Math.round(durationSecs),
      poster_id: posterId,
    }, env);

    return json({ status: 'ok', analysis, poster_id: posterId });

  } catch (err) {
    console.error('Webhook error:', err);
    return json({ status: 'error', message: err.message }, 500);
  }
}

// ════════════════════════════════════════════════════════
// CLAUDE ANALYSIS
// ════════════════════════════════════════════════════════
async function analyseTranscript(transcript, env) {
  const prompt = `Analyse this anonymous voice feedback call transcript.

Extract ONLY:
1. sentiment_score: integer 0–100 (0=very distressed/negative, 50=neutral, 100=very positive/happy)
2. keywords: up to 3 single common words describing main topics (e.g. "work", "family", "stress")
3. themes: up to 2 category labels from this fixed list only:
   Wellbeing | Work & Career | Community | Relationships | Environment | Health | Finances | Future

Rules:
- Never extract names, locations, ages, or any identifying detail
- Keywords and themes must be general — never specific to the individual
- Respond ONLY with valid JSON, no markdown, no explanation

Transcript:
"""
${transcript.slice(0, 2000)}
"""

Respond with exactly:
{"sentiment_score":<0-100>,"keywords":["word1","word2","word3"],"themes":["Theme1","Theme2"]}`;

  const res = await fetch('https://api.anthropic.com/v1/messages', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'x-api-key': env.ANTHROPIC_API_KEY,
      'anthropic-version': '2023-06-01',
    },
    body: JSON.stringify({
      model: 'claude-haiku-4-5-20251001',
      max_tokens: 120,
      messages: [{ role: 'user', content: prompt }],
    }),
  });

  if (!res.ok) throw new Error(`Claude API error: ${res.status}`);

  const data   = await res.json();
  const text   = data.content?.[0]?.text || '';
  const clean  = text.replace(/```json|```/g, '').trim();
  const parsed = JSON.parse(clean);

  const validThemes = new Set([
    'Wellbeing','Work & Career','Community','Relationships',
    'Environment','Health','Finances','Future'
  ]);

  return {
    sentiment_score: Math.min(100, Math.max(0, Math.round(parsed.sentiment_score))),
    keywords: (parsed.keywords || []).slice(0,3).map(k => k.toLowerCase().trim()),
    themes:   (parsed.themes   || []).slice(0,2).filter(t => validThemes.has(t)),
  };
}

// ════════════════════════════════════════════════════════
// SUPABASE WRITE
// ════════════════════════════════════════════════════════
async function writeToSupabase({ sentiment_score, keywords, themes, duration_seconds, poster_id }, env) {
  const res = await fetch(`${env.SUPABASE_URL}/rest/v1/calls`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      apikey: env.SUPABASE_SERVICE_KEY,
      Authorization: `Bearer ${env.SUPABASE_SERVICE_KEY}`,
      Prefer: 'return=minimal',
    },
    body: JSON.stringify({
      sentiment_score,
      keywords,
      themes,
      duration_seconds,
      poster_id,          // null if no poster in URL — that's fine
      // No transcript, no user data, no IP address stored
    }),
  });

  if (!res.ok) {
    const err = await res.text();
    throw new Error(`Supabase write failed: ${err}`);
  }
}

// ════════════════════════════════════════════════════════
// HELPERS
// ════════════════════════════════════════════════════════
function json(data, status = 200) {
  return new Response(JSON.stringify(data), {
    status,
    headers: { 'Content-Type': 'application/json', ...corsHeaders() },
  });
}

function corsHeaders() {
  return {
    'Access-Control-Allow-Origin': '*',
    'Access-Control-Allow-Methods': 'POST, GET, OPTIONS',
    'Access-Control-Allow-Headers': 'Content-Type, x-webhook-secret',
  };
}
