-- ════════════════════════════════════════════════════
-- FEEDBACK.ZONE — Supabase Schema
-- Run this in your Supabase SQL editor
-- ════════════════════════════════════════════════════

create table public.calls (
  id               uuid primary key default gen_random_uuid(),
  created_at       timestamptz not null default now(),
  sentiment_score  integer not null check (sentiment_score between 0 and 100),
  keywords         text[]  not null default '{}',
  themes           text[]  not null default '{}',
  duration_seconds integer not null default 0,
  poster_id        text    -- e.g. 'north-quarter', 'riverside'. NULL if no poster ID in URL.
);

-- Indexes for common query patterns
create index calls_created_at_idx on public.calls (created_at desc);
create index calls_poster_id_idx  on public.calls (poster_id);

-- Row level security
alter table public.calls enable row level security;

-- Public dashboard can read (last 24h only — dashboard never needs older data)
create policy "Public read last 24h"
  on public.calls for select
  using (created_at >= now() - interval '24 hours');

-- Worker (service key) can insert
create policy "Service role insert"
  on public.calls for insert
  with check (true);

-- ── Optional: auto-purge rows older than 30 days ─────
-- Enable pg_cron in Supabase dashboard first, then uncomment:
-- select cron.schedule('purge-old-calls','0 3 * * *',
--   $$ delete from public.calls where created_at < now() - interval '30 days' $$);
