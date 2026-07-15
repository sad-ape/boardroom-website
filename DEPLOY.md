# Deploy this site

The live site is assembled straight from `design-source/` by a GitHub Actions
workflow (`.github/workflows/deploy.yml`) — no build tools, no Node, no bundler.
Every push to `main` redeploys automatically.

Currently live at: https://sad-ape.github.io/boardroom-website/

## Deploy your own copy (GitHub Pages — free)

1. **Fork** this repo to your GitHub account (top-right "Fork" button).
2. In your fork, open the **Actions** tab and click **"I understand my
   workflows, enable them"** (forks start with Actions disabled).
3. Go to **Settings → Pages → Build and deployment** and set
   **Source = GitHub Actions**.
4. Open the **Actions** tab → **"Deploy to GitHub Pages"** → **Run workflow**
   (or just push any commit).
5. When it finishes (~30s), your site is live at
   `https://<your-username>.github.io/<repo-name>/`.

After that, editing `design-source/Boardroom.dc.html` and pushing to `main`
redeploys on its own.

## What actually gets served

The workflow copies these into the published site:

- `design-source/Boardroom.dc.html` → `index.html`
- `design-source/support.js` → `support.js`
- `assets/` → `assets/`
- `assets/stickers/*.png` → `uploads/sticker/*.png` (the sticker picker looks
  for them there)

React is loaded at runtime from a public CDN (unpkg), so the page just needs to
be served over http(s) — it won't render by double-clicking the file locally.

## Any other static host (Netlify, Cloudflare Pages, Vercel, S3, …)

Serve a folder laid out exactly like the list above (index.html + support.js +
assets/ + uploads/sticker/). You can produce it locally with:

```sh
mkdir -p site/assets site/uploads/sticker
cp design-source/Boardroom.dc.html site/index.html
cp design-source/support.js        site/support.js
cp -r assets/*                     site/assets/
cp assets/stickers/*.png           site/uploads/sticker/
```

Then point the host at `site/`.
