# Stacks — a local reader

A single-page app that reads EPUB, PDF, and TXT files entirely in your browser.
No server, no accounts, no CDN — everything (including the PDF/EPUB libraries)
ships locally in this repo. Your books are stored on-device (IndexedDB) and
never leave it.

## Run it on your iPhone (or anywhere) via GitHub Pages

1. Create a new GitHub repo and upload everything in this folder
   (`index.html` + the `vendor/` folder), keeping the same structure.
2. In the repo: **Settings → Pages → Source → Deploy from branch → main / (root)** → Save.
3. GitHub gives you a URL like `https://yourname.github.io/reponame/`.
4. Open that URL in Safari on your iPhone.
5. Tap **Share → Add to Home Screen** — it now opens full-screen like an app.

## Using it

- **Add books**: tap "+ Add books" or drag files onto the window (desktop).
- Supports `.epub`, `.pdf`, `.txt`.
- Your shelf, reading position, and progress % are saved automatically.
- Font size (A− / A+) and Day/Night theme are in the top toolbar.
- Arrow keys turn pages (epub/pdf).

## Notes

- Everything runs client-side — books are stored in the browser's IndexedDB,
  scoped to whichever URL you open the app from. If you clear Safari's site
  data or run very low on storage, iOS may evict it, so it's not meant as
  permanent archival storage — just a fast personal reader.
- No network calls are made at runtime; the `vendor/` folder contains the
  PDF.js, JSZip, and epub.js libraries so the app works fully offline once loaded.
