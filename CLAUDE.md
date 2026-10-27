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

## Secretary sheet (Google Sheets, two-way, approval-based)
- Sheet "Studio Manager – מזכירה" in yasharulpan's Drive, id `window.SM_SHEET_ID` (firebase-config.js).
- Separate Google token, scope `spreadsheets` only (`window.__sheetAuth()` redirect with state `sh…`, stored in `studio-sheet-tok`) —
  never touches the calendar token/scope. Google Sheets API must be enabled in project studio-manager-7eb85.
- Full sync (tabs + "נתונים" rewrite + requests) on app open and on the "📋 סנכרון מזכירה" button. Light poll of "בקשות" every 30s while
  visible and on every return to the app (`secPoll`) → red count badge (unique pending request ids) on the "📋 סנכרון מזכירה" button,
  which sits next to the calendar sync button (`gsyncUI`); tapping it opens the approval popup (no auto-popup).
  Approve → execute, mark בוצע, rewrite "נתונים", then calendar sync (`runSyncRef`). Values may start with ' (stripped).
- Tab "נתונים": rewritten each sync, one row per gig (`SM_HEAD`). Tab "בקשות": A–G = id בקשה, פעולה (הוספה/עדכון/מחיקה), id אירוע,
  שדה (a "נתונים" header name), ערך, הערה, סטטוס; app writes H–I (date, reply) and the new id into C on add. Several rows with the same
  request id = one request. Pending (status empty/ממתין) are shown in `SecretaryInbox`; only on approval: cloud backup
  (`__studioBackups.snap`, slots backup-pre-0..9) → `rqApply` → status בוצע / נדחה. Executed ids kept in `config.rqDone`.
- New gig fields: `dress` (קוד לבוש), `production` (הפקה), `setlist` (רשימת שירים).
