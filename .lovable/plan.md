# FURSAD MVP Plan

A privacy-first PWA for Wajir youth: anonymous incident reporting, mentorship requests, opportunities board, and a forum. Green-on-light design, no emojis, trilingual UI, voice pitch-shift, admin panel.

## Scope of this build

I will build the full MVP in one repo using the project's existing stack (TanStack Start + Lovable Cloud / Supabase under the hood). I'll deliver a working, installable PWA with all 5 main tabs, status check, admin panel, language toggle, voice recording with client-side pitch shift, Leaflet map picker (area name only), and end-to-end privacy guarantees.

## Stack mapping (vs your spec)

Your spec says "React + Tailwind + Node/Express + Postgres + Supabase". The Lovable project is **TanStack Start (React 19) + Tailwind v4 + Lovable Cloud (Supabase Postgres + Edge)**. Functionally equivalent — I'll use it instead of standing up a second Express server, which would not run on this platform.

- **Frontend:** TanStack Start, Tailwind v4, shadcn/ui, Leaflet, Web Audio API
- **Backend:** TanStack server functions + Lovable Cloud (Supabase Postgres, RLS, Storage, Edge for SMS)
- **SMS:** Twilio via Lovable connector (one-shot send, number deleted right after)
- **PWA:** manifest + installability (no offline service worker by default — can add later if you want true offline)

## Data model (no user table)

```
reports(id, session_id_hash, incident_type, description, voice_url, area_name, when_bucket, verified, created_at)
mentorship_requests(id, session_id_hash, need, contact_method, phone_encrypted, consent, status, sent_at, created_at, expires_at)
opportunities(id, title, type, opp_date, location, description, featured, created_at)
forum_posts(id, nickname, category, content, voice_url, upvotes, created_at)
forum_replies(id, post_id, nickname, content, created_at)
blocked_keywords(id, keyword)
admins(id, username, password_hash)  -- admin login only
```

RLS: public can INSERT reports / mentorship / posts / replies, public can SELECT opportunities + posts + verified reports. Admin operations go through server functions gated by an admin session cookie.

## Routes

```
/                      Home (verified reports + featured opps)
/report                Anonymous report form (image-matched layout)
/status                Session ID status check
/mentors               Mentorship request form
/opps                  Opportunities (All / Saved tabs, saved via localStorage)
/discuss               Forum (category tabs, create post, upvote, reply)
/admin                 Admin login + dashboard (reports, mentorship, opps, forum)
```

Bottom nav (5 items, icon + label, no emojis) on all public routes. Top bar with FURSAD wordmark + language toggle.

## Privacy implementation

- Session IDs: 6-char base32 shown to user; only SHA-256 hash stored
- Voice: recorded → pitch-shifted in browser via Web Audio (offline render) → user previews → uploads altered blob only; original never leaves device
- Phone numbers: AES-256-GCM encrypted at rest using server-side key; deleted immediately after Twilio send; daily cleanup of anything >7 days old
- Map: Leaflet pin used to derive area name (reverse geocode or manual entry); coordinates discarded before save
- No analytics, no cookies beyond the admin session, no third-party scripts

## i18n

Lightweight in-app dictionary (`en` / `sw` / `so`) via React context — instant switch, persisted to localStorage. Covers all UI strings; user content stays in original language.

## Admin panel

Hardcoded seed admin (username + bcrypt hash via env). Sets an httpOnly session cookie. Pages: Reports (verify toggle), Mentorship (send SMS button → deletes phone), Opportunities (CRUD + featured toggle), Forum (delete posts, manage blocked keywords).

## Accessibility

Semantic HTML, ARIA labels on all icon buttons, "Listen" button per screen using `speechSynthesis`, focus-visible rings, 16px+ text, high-contrast tokens.

## Design tokens (src/styles.css)

- `--background: #F7F9FC`, `--card: #FFFFFF`, `--foreground: #1F2937`, `--muted-foreground: #6B7280`
- `--primary: #10B981` (with darker `#2E7D32` variant for verified badge)
- 12px card radius, subtle shadow
- Icons from `lucide-react` (already installed) — no emojis anywhere

## Things I will NOT build (matches your spec)

No user accounts, no coordinator UI, no elder dashboard, no auto-matching, no image/video uploads, no private messaging, no analytics.

## Deliverables

- Working app at the preview URL, installable PWA
- Admin login: I'll set demo creds via secrets and share them
- Twilio: requires connecting your Twilio account — I'll prompt for that when wiring the SMS send (the rest works without it)
- A short README section in the repo with deploy notes

## Confirm before I start

1. **Twilio**: connect now, or stub the SMS send and you connect later?
2. **Admin credentials**: pick a username, or use `admin` with a generated password I show you once?
3. **Offline PWA**: manifest-only (installable) is the default — say the word if you want full offline cache too.
