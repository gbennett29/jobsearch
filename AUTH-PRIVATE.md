# Keeping your hosted CRM private (Firebase)

## How Firebase “privacy” works

- **Firebase Hosting** serves your `index.html` to anyone who knows the URL. That’s normal—the HTML/JS is not your secret layer.
- **Privacy** = only **you** (or people you allow) can **read/write** data in Firestore (and use the app in a meaningful way).

So: **turn on Authentication + strict Firestore rules.** Without a signed-in user matching your rule, Firestore returns **permission denied**—the app stays empty for strangers.

---

## Step 1 — Google sign-in (Console)

1. [Firebase Console](https://console.firebase.google.com/) → your project → **Build → Authentication**
2. **Get started** → **Sign-in method** → enable **Google** (set a support email) → **Save**.
3. **Settings** tab → **Authorized domains**: ensure your **Hosting** domain is listed (e.g. `yourproject.web.app`, `yourproject.firebaseapp.com`). For testing from your laptop, add **`localhost`**.
4. Deploy the updated `index.html` / run locally, open the app, click **Sign in with Google**.

### Optional: allowlist in the app code

In `index.html`, `ALLOWED_EMAILS` is an empty array by default (any Google user who passes **Firestore rules** can use the app). To block extra accounts in the UI as well, set:

`const ALLOWED_EMAILS = ['your.real.email@gmail.com'];`

Firestore rules should still match the same email(s).

---

## Step 2 — Firestore rules (lock to you)

**Firestore → Rules**, replace permissive rules with something like this (use **your real email**):

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    function isYou() {
      return request.auth != null
        && request.auth.token.email == "YOUR_EMAIL@gmail.com";
    }

    match /opportunities/{docId} {
      allow read, write: if isYou();
    }

    match /jobFeed/{docId} {
      allow read, write: if isYou();
    }

    match /{document=**} {
      allow read, write: if false;
    }
  }
}
```

- **Publish** rules.
- If you use a provider where `email` isn’t on the token, use `request.auth.uid` and your UID from the Auth user list instead.

**Until someone signs in**, Firestore will deny reads/writes—expected.

---

## Step 3 — (Optional) App Check

**App Check** reduces abuse (bots hammering your project). Not a substitute for rules, but useful once auth is on.  
Firebase Console → **App Check** → register your web app.

---

## Step 4 — Sign-in in the web app (done in `index.html`)

The hosted app loads **Firebase Auth (compat)** and shows a **Sign in with Google** gate until `onAuthStateChanged` returns a user, then calls `loadFromFirestore()`. Use **Sign out** in the sidebar to clear the session.

---

## Quick checklist

| Goal | Action |
|------|--------|
| Data private | Auth on + Firestore rules like above |
| URL not guessable | OK to use Hosting URL; optional custom domain |
| Strangers see empty app | Rules deny reads without auth |

---

## If you’re stuck on “Permission denied” after changing rules

1. Sign in with Google on the app first.  
2. Confirm your **email** in the rules matches your Google account exactly.  
3. Add **localhost** to Authorized domains if testing from `file://` or a local server.

## If Google popup says “unauthorized domain”

Add your exact origin (e.g. `localhost` or `yoursite.web.app`) under **Authentication → Settings → Authorized domains**.
