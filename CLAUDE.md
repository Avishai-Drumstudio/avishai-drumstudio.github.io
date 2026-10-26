# avishai-drumstudio.github.io

Owner: Avishai Yashar, freelance drummer. Talk to him in Hebrew. He usually works from his phone,
so he asks for a change in chat and expects it live on the site (push to `main` = deployed by GitHub Pages in ~1 min).

## What lives here
- `index.html`, `admin.html`, `assets/` — the landing page (AY | Live & Studio Drums).
- `studio/` — **Studio Manager**, his private app for gigs, calendar, payments, receipts, clients and subs.
  URL: https://avishai-drumstudio.github.io/studio/

## Studio Manager (`studio/`)
- `studio/index.html` is the whole app in one file: React 18 UMD + htm, Hebrew RTL, Rubik font. Styles: Tailwind classes
  precompiled into `studio/app.css` (no Tailwind CDN, for speed) — after adding new Tailwind classes, regenerate it with the
  Tailwind v3 standalone CLI (content = studio/index.html, theme colors ink/muted/line/glass/glass2/ok/warn/bad = CSS vars,
  font Rubik) and bump the `app.css?v=` query.
  Edit this file directly — it is the source of truth.
- `studio/firebase-config.js` — Firebase web config (`window.FIREBASE_CONFIG`) and the owner Google accounts
  (`window.STUDIO_OWNERS`). The config is not secret; security is in Firestore rules.
- `studio/firestore.rules` — copy of the rules pasted in the Firebase console (per-user data under `users/{uid}/data/*`; approved friends in `config/allowed.emails`, managed by owners in Settings → חברים מורשים).
- The script block at the top of `index.html` (after the Firebase SDK tags) is the "gate": Google sign-in, and a
  `window.claude.use()` shim so the app code (written originally for a Claude artifact) runs unchanged:
  - `db` → Firestore docs `users/{uid}/data/{studio | gigs-YYYY-1|2 | phonebook}` (every user has his own data)
  - `user`, `downloads` → simple local implementations
  - `mcp` → Google Calendar REST. Calendar access via Google Identity Services token client (`window.GOOGLE_CLIENT_ID`
    in firebase-config.js; origin https://avishai-drumstudio.github.io is authorized on that OAuth client), token ~1h in
    localStorage; the calendar account may differ from the login account (Avishai: login avishy407, calendar yasharulpan).
    Permanent option ("bridge"): Settings → יומן גוגל → חיבור קבוע — a Google Apps Script web app in the calendar
    account proxies Calendar REST calls (secret + url stored in `config.gbridge`); when set, the app uses it instead of GIS
    tokens and never needs to reconnect. Calendar sync is manual only (no auto-sync — Avishai asked).
    Tools emulated: `list_calendars`, `create_event` (first links to an existing event same day within 60 min instead of
    duplicating), `update_event`, `delete_event`.
  - Login once per device: the last user is cached in localStorage and the app opens instantly, auth is verified in the background.
  - `sample` is not available here (the in-app AI "secretary" and quick-add are hidden).
- Data model (see `normalize()` in the app): gigs split into half-year docs to keep each doc small; `config`
  holds types/colors/texts/todos; `gsync` maps gig id → Google Calendar event id.
- Before pushing: extract the main `<script>` and run `node --check` on it; keep Hebrew UI text natural.
- On every change to the app, bump the version in BOTH `studio/version.txt` and `window.STUDIO_VERSION` in
  `studio/index.html` (same value, e.g. YYYYMMDDHHMMSS). Open apps poll version.txt and reload to `?v=<new>`, which
  bypasses GitHub Pages' 10-minute cache — so changes show up on his phone right away.

## Commits
End commit messages with the attribution lines the session gives you.

## Secretary sheet (Google Sheets, two-way)
- Sheet "Studio Manager – מזכירה" in yasharulpan's Drive, id in `window.SM_SHEET_ID` (firebase-config.js). Uses the same Google token
  as the calendar (scope calendar + spreadsheets, `window.__gToken()`); Google Sheets API must be enabled in project studio-manager-7eb85.
- Tab "אירועים": rewritten from the app on every change (when a token is active). Tab "בקשות מהמזכירה": read on open / sync;
  columns A–I = מזהה בקשה, פעולה, id אירוע, שדה, ערך, פרטים (JSON), סטטוס, תאריך ביצוע, הערה. Actions: הוספת אירוע · עדכון שדה ·
  סימון תשלום · צירוף מסמך (see `rqApplyAll`). Executed request ids are kept in `config.rqDone` so a row never runs twice.
- Documents: `config.docs` = [{number (key), type, date, amount, url, gigIds}] — one document can cover several events; panel "📎 מסמכים" in event details.

