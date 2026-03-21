# Updates from this Cursor session (reference)

Use this if you need to re-apply or verify features after a revert.

| Feature | Where |
|--------|--------|
| **Notion-like light UI** | **`index.html` only** — no duplicate HTML file |
| **Job feed** | Sidebar “Job feed”, Firestore `jobFeed`, “+ Save job”, promote to pipeline |

**Single entry point:** Edit and deploy **`index.html`** only.

**Firebase Hosting:** `/` serves `index.html` by default.

**Firestore dev rules (temporary, wide open):** `firestore.rules` + `firebase.json` + `.firebaserc`. Deploy with `firebase deploy --only firestore:rules` (requires [Firebase CLI](https://firebase.google.com/docs/cli) and `firebase login`). Revert rules before a public URL.

**Separate project:** Company analysis CLI lives under `~/Cursor` (`company_agent`), not in this `jobsearch` folder.

**Firebase web config:** Not in `index.html`. Copy `.env.example` → `.env`, run `node scripts/generate-firebase-config.mjs` to create gitignored `firebase-config.js`. Regenerate before `firebase deploy` if Hosting needs it.
