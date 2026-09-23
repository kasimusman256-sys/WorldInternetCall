# World Internet Call

A mobile-first prototype for a call-over-internet app. Built with React +
TypeScript + Vite, structured to plug into Supabase for auth/data and
Africa's Talking Voice API for real calling — neither is required to run
the app today.

**This is a prototype.** It does not place real phone calls. The call
screen honestly shows "Connecting..." and then "Failed" with an
explanation, because no voice backend is wired up yet. See
`src/services/voiceService.ts` for exactly where and how the real
integration will plug in.

## Tech stack

- React 18 + TypeScript + Vite
- react-router-dom for navigation
- @supabase/supabase-js (client is created either way; falls back to
  local-only mock data when no Supabase project is configured)
- Plain CSS (no UI framework), dark theme, mobile-first

## Project structure

```
world-internet-call/
├── .env.example          # copy to .env, fill in real values
├── .gitignore
├── index.html
├── package.json
├── tsconfig.json
├── tsconfig.node.json
├── vite.config.ts
└── src/
    ├── main.tsx           # app entry, router setup
    ├── App.tsx             # route definitions
    ├── index.css           # global dark theme styles
    ├── vite-env.d.ts       # typed env vars
    ├── types/
    │   └── index.ts        # CallStatus, CallRecord, UserProfile, etc.
    ├── lib/
    │   └── supabaseClient.ts   # single Supabase client + config check
    ├── services/
    │   ├── voiceService.ts     # ★ call provider abstraction (see below)
    │   ├── authService.ts      # register/login/logout wrapper
    │   └── callHistoryService.ts  # read/write call records
    ├── context/
    │   └── AuthContext.tsx     # current user + session state
    ├── components/
    │   ├── PhoneInput.tsx      # number input with country code select
    │   ├── CallStatusBadge.tsx
    │   ├── CreditsDisplay.tsx
    │   ├── RecentCalls.tsx
    │   └── ProtectedRoute.tsx  # redirects to /login if signed out
    └── pages/
        ├── Landing.tsx
        ├── Login.tsx
        ├── Register.tsx
        ├── Dashboard.tsx       # phone input, call button, history
        └── Profile.tsx
```

## Why it works without a backend

Every screen is usable immediately after `npm install && npm run dev`,
with no Supabase project required:

- **Auth**: if `VITE_SUPABASE_URL`/`VITE_SUPABASE_ANON_KEY` aren't set
  (or still contain the placeholder values from `.env.example`),
  `authService` and `AuthContext` fall back to a local mock session
  stored in `localStorage`. Any email/password "creates" a local account.
- **Call history**: `callHistoryService` falls back to `localStorage`
  the same way.
- **Calling**: `voiceService.ts` always uses `MockVoiceProvider`, which
  never fakes success — it reports "connecting" then "failed" with a
  clear reason.

Once you connect a real Supabase project, auth and call history switch
over automatically (see "Connecting Supabase" below). Voice calling stays
off until you implement `AfricasTalkingVoiceProvider`.

## Running locally (from Android / GitHub-based workflow)

You don't need a full desktop dev setup. A common flow from an Android
phone:

1. Push this project to a GitHub repo (see "Getting this onto GitHub"
   below).
2. Use a cloud dev environment that can open a GitHub repo and run
   Node.js commands (for example GitHub Codespaces, or any similar
   browser-based VS Code / terminal environment). Open the repo there.
3. In its terminal:
   ```bash
   npm install
   npm run dev
   ```
4. Open the forwarded preview URL it gives you — that's the app running
   live, viewable from your phone's browser.

For a build you can deploy as a static site (e.g. Vercel, Netlify,
Cloudflare Pages, GitHub Pages):

```bash
npm run build   # outputs to dist/
npm run preview # locally preview the production build
```

Any static host that lets you connect a GitHub repo will build and
deploy this automatically on every push once configured.

## Getting this onto GitHub

1. Create a new empty repository on GitHub (no README/license, so there's
   no conflict).
2. From the environment where you have these files (or via GitHub's
   "upload files" web UI, which works fine from a phone browser for an
   initial commit):
   ```bash
   git init
   git add .
   git commit -m "Initial prototype"
   git branch -M main
   git remote add origin https://github.com/YOUR_USERNAME/YOUR_REPO.git
   git push -u origin main
   ```
3. `.env` is git-ignored on purpose — never commit real secrets. Only
   `.env.example` (placeholders) is committed.

## Connecting Supabase (auth + database)

1. Create a project at supabase.com.
2. Copy `.env.example` to `.env` and fill in `VITE_SUPABASE_URL` and
   `VITE_SUPABASE_ANON_KEY` from Project Settings → API. These are the
   public anon key — safe for a frontend build, **as long as Row Level
   Security (RLS) is enabled** on every table.
3. Suggested tables:
   - `profiles` (id uuid references auth.users, display_name text,
     credits_balance numeric default 0, created_at timestamptz default now())
   - `call_records` (id text primary key, user_id uuid references
     auth.users, phone_number text, status text, started_at timestamptz,
     ended_at timestamptz, duration_seconds int, failure_reason text)
4. Enable RLS on both tables and add policies so a user can only
   read/write rows where `user_id = auth.uid()`.
5. Once these env vars are set, `authService` and `callHistoryService`
   automatically switch from local mock data to real Supabase calls —
   no other code changes needed.

## Connecting Africa's Talking Voice (future work)

**Do not put the Africa's Talking API key in this frontend project.**
Voice APIs require a secret key that must never ship in browser
JavaScript. The plan, already scaffolded in `voiceService.ts`:

1. Build a small backend endpoint (a Supabase Edge Function is a natural
   fit, since Supabase is already in this stack) that holds the Africa's
   Talking API key as a server-side secret and calls their Voice API.
2. Implement `AfricasTalkingVoiceProvider` in `voiceService.ts` to call
   that backend endpoint instead of throwing "not implemented".
3. Set `VITE_VOICE_BRIDGE_URL` in `.env` to that endpoint's public URL
   (the URL is fine to expose — the secret key lives only on the
   backend).
4. Switch `getVoiceProvider()` to return `AfricasTalkingVoiceProvider`
   when `VITE_VOICE_BRIDGE_URL` is set (the switch is already commented
   in that file).
5. Only after this is tested end-to-end should the UI's "prototype /
   not connected" messaging be removed.

## What's intentionally not built yet

- Real calling (see above)
- Payment processing / buying credits
- Push notifications
- Admin/analytics dashboard

## License

Add a license of your choice before making the repo public.
