# RunRoute 🏃

**A daily running route generator.** Different route every day, no highways, Google Maps integration. Install as a PWA on iOS and Android.

---

## Deploy to GitHub Pages (5 minutes)

### Step 1 — Create a GitHub repo

1. Go to [github.com](https://github.com) → **New repository**
2. Name it `runroute` (or anything you like)
3. Set it to **Public** (required for free GitHub Pages)
4. Click **Create repository**

### Step 2 — Upload these files

You can do this two ways:

**Option A — Drag & drop in the browser (easiest):**
1. Open your new repo on GitHub
2. Click **"uploading an existing file"** link
3. Drag the entire contents of this folder in:
   ```
   index.html
   manifest.json
   sw.js
   icons/
     icon-192.png
     icon-512.png
   ```
4. Commit with message: `Initial deploy`

**Option B — Git CLI:**
```bash
git init
git remote add origin https://github.com/YOUR_USERNAME/runroute.git
git add .
git commit -m "Initial deploy"
git push -u origin main
```

### Step 3 — Enable GitHub Pages

1. In your repo, go to **Settings → Pages**
2. Under **Source**, select **Deploy from a branch**
3. Branch: `main` / Folder: `/ (root)`
4. Click **Save**

GitHub will give you a URL like:
```
https://YOUR_USERNAME.github.io/runroute/
```

It takes ~1–2 minutes to go live the first time.

---

## Install as a PWA on your phone

**iPhone (Safari only — Chrome on iOS won't show install):**
1. Open your GitHub Pages URL in **Safari**
2. Tap the **Share button** (box with arrow)
3. Tap **"Add to Home Screen"**
4. Name it "RunRoute" → tap **Add**

**Android (Chrome):**
1. Open your GitHub Pages URL in **Chrome**
2. Tap the **three-dot menu**
3. Tap **"Add to Home Screen"** or **"Install app"**
4. Confirm → it appears on your home screen

---

## Google Maps API Key

The app needs a Google Maps API key. Get one free at:
👉 https://console.cloud.google.com

Enable these APIs on your key:
- Maps JavaScript API
- Directions API
- Places API
- Geocoding API

All have a generous free tier (up to $200/month free credit).

> **Security tip:** Once you're done developing, restrict your API key to your GitHub Pages domain in the Google Cloud Console under **API restrictions → HTTP referrers**.

---

## Making changes

Since it's all one `index.html` file, just edit it and push:
```bash
git add index.html
git commit -m "Your change description"
git push
```
GitHub Pages redeploys automatically within ~30 seconds.

---

## File structure

```
runroute/
├── index.html       ← The entire app (edit this)
├── manifest.json    ← PWA metadata (name, colors, icons)
├── sw.js            ← Service worker (offline caching)
└── icons/
    ├── icon-192.png ← App icon (home screen, small)
    └── icon-512.png ← App icon (splash screen, large)
```
