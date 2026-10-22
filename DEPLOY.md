# TBP Treasury — Setup & Deploy Guide

This is a standalone website, fully private: it opens to a login page, and only
the approved chapter officers can sign in to see or edit the shared live data
(Firebase Firestore). Nobody without an officer account can view anything.

Files that matter. Everything the website serves is in the `src/` folder:
- `src/index.html` — the page markup and app script.
- `src/styles.css` — all of the site's styling (linked from `index.html`).
- `src/firebase-config.js` — your Firebase project's connection settings. **Not in git**
  (gitignored); create it from `src/firebase-config.example.js`. It must be included
  when you deploy, or the site shows "Firebase isn't configured".
- `src/assets/images/tbp-logo.png`, `tbp-logo-mark.png` — the watermark and badge logos.
- `firestore.rules` (top level) — the security rules you paste into Firebase (not deployed to Vercel).

Config is already filled in — you just publish the rules and deploy. ~10 minutes.

---

## Step 1 — Config (already done ✅)

Your project's Firebase settings are in `src/firebase-config.js` (already filled in on
the treasurer's machine; not committed to GitHub). They're IDs, not secrets.

Starting funds (e.g. Line Budget, Induction Money) are **not** set in `index.html` —
real dollar amounts don't belong in a static file anyone can view via "View
Source". Add them live from the **Budgets tab** after you sign in as an
officer; they're stored only in Firestore (`budgets/main`).

**Who can sign in** is defined by the `officers` collection in Firestore — one
document per officer, whose **document ID is their email in lowercase**. No
emails live in `index.html` or `firestore.rules`, so both are safe to keep in a
public repository. The app decides whether to show the dashboard by asking the
database, not by carrying the list.

---

## Step 2 — Add the officers, then publish the security rules

Do these **in this order**. The rules only let in emails listed in `officers`,
so publishing the rules before the list exists locks everyone out.

**2a. Create the officer list**
1. Go to **console.firebase.google.com** → your `tbp-treasury` project.
2. **Build → Firestore Database → Data** tab → **+ Start collection**.
3. Collection ID: `officers`.
4. Document ID: the officer's email, **all lowercase** (e.g. `jane.doe@stonybrook.edu`).
   Add one field so the document saves, e.g. `name` (string) = `Jane Doe`.
5. Save, then **+ Add document** for each remaining officer the same way.

**2b. Publish the rules**
1. Same project → **Build → Firestore Database → Rules** tab.
2. Delete what's there, paste the full contents of `firestore.rules`.
3. Click **Publish**.
4. Sign in to the site with an officer account to confirm you still get in.

---

## Step 3 — Deploy the site to Vercel

Easiest path (no command line):

1. Go to **vercel.com** and sign up / log in (use "Continue with GitHub" or email).
2. On your dashboard click **Add New… → Project**.
3. Choose **Deploy a template / import** — or the simplest option:
   drag the **`src` folder** (not the whole project) onto the Vercel deploy area.
   - *Command-line option:* run `npx vercel` from inside the `src` folder.
   - Connecting the GitHub repo instead won't work as-is: `firebase-config.js`
     isn't in GitHub, so the deployed site would show "Firebase isn't configured".
4. When it asks for settings, no build step is needed — it's plain HTML.
   Framework preset: **Other**. Root directory: `src` (the folder with `index.html`).
5. Click **Deploy**. In a few seconds you'll get a URL like
   `https://tbp-treasury.vercel.app`.
6. (Optional) In the project's **Settings → Domains** you can rename it, e.g.
   `sbu-tbp-treasury.vercel.app`.

---

## Step 4 — Authorize your Vercel domain in Firebase

Google sign-in only works from approved domains.

1. Firebase console → **Build → Authentication → Settings → Authorized domains**.
2. Click **Add domain** and add your Vercel domain, e.g. `tbp-treasury.vercel.app`.
   (`localhost` is already there for testing.)

---

## Step 5 — Go live

1. Open your Vercel URL — you'll land on the **login page**.
2. Click **Sign in with Google** → choose an approved officer account.
   The dashboard appears. (A non-officer account is refused with a message and
   never sees the data.)
3. Add your first transaction. Open the same URL on another officer's device —
   the entry shows up live.
4. Share the URL with the board's officers. Everyone must sign in; only accounts
   listed in `officers` get in.

---

## Updating the site later

- **Add/remove an officer:** in the Firebase console, add or delete their document
  in the `officers` collection (Step 2a). Takes effect immediately — no rules
  re-publish and no site redeploy. Officers can't change this list from the app.
- **Change colors or layout:** edit `src/styles.css`, then re-deploy the same way.
- **Change categories:** edit `src/index.html`, then re-deploy on Vercel (drag the
  `src` folder again).
- **Change starting funds, budgets, or add transactions:** just do it in the app
  (Budgets tab); no redeploy needed — it's all stored in Firestore.

## Notes on cost & longevity
- Firebase Spark (free) tier and Vercel Hobby (free) tier easily cover a club.
- Use a chapter-owned Google account as the Firebase owner so access survives
  officer turnover. Add/remove officers in the `officers` collection.
