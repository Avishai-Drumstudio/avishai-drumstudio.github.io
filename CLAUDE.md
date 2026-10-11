# avishai-drumstudio.github.io

Owner: Avishai Yashar, freelance drummer. Talk to him in Hebrew. He usually works from his phone,
so he asks for a change in chat and expects it live on the site (push to `main` = deployed by GitHub Pages in ~1 min).

## What lives here
- `index.html`, `admin.html`, `assets/` — the landing page (AY | Live & Studio Drums).
- `studio/` — **Studio Manager**, his private app for gigs, calendar, payments, receipts, clients and subs.
  URL: https://avishai-drumstudio.github.io/studio/

## Studio Manager (`studio/`)
- `studio/index.html` is the whole app in one file: React 18 UMD + htm + Tailwind CDN, Hebrew RTL, Rubik font.
  Edit this file directly — it is the source of truth.
- `studio/firebase-config.js` — Firebase web config (`window.FIREBASE_CONFIG`) and the owner Google accounts
  (`window.STUDIO_OWNERS`). The config is not secret; security is in Firestore rules.
- `studio/firestore.rules` — copy of the rules pasted in the Firebase console (per-user data under `users/{uid}/data/*`; approved friends in `config/allowed.emails`, managed by owners in Settings → חברים מורשים).
- The script block at the top of `index.html` (after the Firebase SDK tags) is the "gate": Google sign-in, and a
  `window.claude.use()` shim so the app code (written originally for a Claude artifact) runs unchanged:
  - `db` → Firestore docs `users/{uid}/data/{studio | gigs-YYYY-1|2 | phonebook}` (every user has his own data)
  - `user`, `downloads` → simple local implementations
  - `mcp` → Google Calendar REST (separate Google sign-in with calendar scope, token in localStorage);
    tools emulated: `list_calendars`, `create_event`, `update_event`, `delete_event`
  - `sample` is not available here (the in-app AI "secretary" and quick-add are hidden).
- Data model (see `normalize()` in the app): gigs split into half-year docs to keep each doc small; `config`
  holds types/colors/texts/todos; `gsync` maps gig id → Google Calendar event id.
- Before pushing: extract the main `<script>` and run `node --check` on it; keep Hebrew UI text natural.

## Commits
End commit messages with the attribution lines the session gives you.
