# 28th Street — Deployment Guide

This is a standalone Vite + React project. It won't fully work inside a Claude
artifact preview because artifacts block outbound network requests — the app
needs real network access to reach Firestore (for data) and EmailJS (for
notification emails). Run it locally or deploy it to get a working site.

## 1. Run it locally

```bash
npm install
npm run dev
```

Opens at `http://localhost:5173`. The dashboard will load, but saving/loading
data won't work until you configure Firebase (step 2).

## 2. Configure Firebase (required for data to persist)

The app talks directly to Firestore's REST API — no Firebase SDK needed.

1. Go to [console.firebase.google.com](https://console.firebase.google.com) and create a project.
2. Enable **Firestore Database** (Build → Firestore Database → Create database).
3. In Firestore Rules, allow read/write to the `kv` collection (tighten this later if you want real auth):
   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /kv/{doc} {
         allow read, write: if true;
       }
     }
   }
   ```
4. Project settings → General → "Your apps" → add a Web app → copy the config values.
5. Open `src/App.jsx`, find the `firebaseConfig` object near the top, and paste in
   your real `apiKey`, `projectId`, etc. (replace the `PASTE_YOUR_...` placeholders).

## 3. Configure EmailJS (optional — for tenant email notifications)

Email sending is silently skipped until this is set up.

1. Create a free account at [emailjs.com](https://www.emailjs.com).
2. Add an email service and a template.
3. In `src/App.jsx`, find `EMAILJS_SERVICE_ID`, `EMAILJS_TEMPLATE_ID`, and
   `EMAILJS_PUBLIC_KEY` and fill them in.

## 4. Build for production

```bash
npm run build
```

Outputs static files to `dist/`.

## 5. Deploy

Any static host works since this is a client-only app.

**Vercel**
```bash
npm install -g vercel
vercel
```

**Netlify**
```bash
npm install -g netlify-cli
netlify deploy --prod --dir=dist
```

**Firebase Hosting** (convenient since you already have a Firebase project)
```bash
npm install -g firebase-tools
firebase login
firebase init hosting   # public dir: dist
firebase deploy
```

## Notes

- Owner contact info (`OWNER_PHONE`, `OWNER_EMAIL`, `OWNER_ZELLE`) is set near
  the top of `src/App.jsx` — update if needed.
- The Firestore API key ends up in client-side JS, which is normal for
  Firebase web apps — access is controlled by Firestore security rules, not
  by hiding the key.
- The UI is bilingual (English/Arabic).
