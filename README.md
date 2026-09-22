# WebAR Artwork Viewer — Prototype

A single-file WebAR viewer using MindAR.js (image tracking) + A-Frame (3D/video rendering).
Currently wired to MindAR's public sample target/video so you can verify camera tracking
works on your phone *before* wiring in your own artwork.

## What's in this prototype
- `index.html` — everything: markup, styles, AR scene, and control logic. No build step, no dependencies to install.
- `sample-video.mp4` — a small (~640 KB) self-hosted test video. **Must be uploaded to the same folder as `index.html`** — the page references it by relative path (`./sample-video.mp4`), so it only works once both files sit together in your deployed site.

## 1. Local testing (HTTPS requirement)

Mobile browsers **block camera access on any non-HTTPS origin**, except `http://localhost` on the *same device* you're developing on. Since you need to test on your **phone**, not your laptop, you need HTTPS even locally. Two easy options:

### Option A — `ngrok` (fastest)
```bash
# In the folder containing index.html:
npx serve .          # starts a local server, e.g. on http://localhost:3000

# In a second terminal:
npx ngrok http 3000  # gives you a public https://xxxx.ngrok-free.app URL
```
Open the `ngrok` HTTPS URL on your phone. Done — no cert setup needed.

### Option B — `vite`/`live-server` with a local cert
```bash
npm install -g local-web-server
ws --https -d .
```
This self-signs a cert; your phone will show a security warning you must accept once.

**Either way:** open the URL on your phone's camera-capable browser (Safari on iOS, Chrome on Android), grant camera permission, and point it at a printed copy (or displayed on a second screen) of MindAR's sample "card" target image:

`https://cdn.jsdelivr.net/gh/hiukim/mind-ar-js@1.1.4/examples/image-tracking/assets/card-example/card.png`

Download/open that image, display it full-screen on a laptop or print it, then point your phone's camera at it to confirm tracking + video overlay works.

## 2. Permanent free deployment

### GitHub Pages
```bash
git init
git add index.html README.md
git commit -m "WebAR prototype"
git branch -M main
git remote add origin https://github.com/<you>/<repo>.git
git push -u origin main
```
Then in the repo: **Settings → Pages → Deploy from branch → main → / (root)**.
GitHub Pages serves everything over HTTPS automatically — no extra config needed.
Your live URL: `https://<you>.github.io/<repo>/`

### Vercel (equally simple, faster propagation)
```bash
npm install -g vercel
vercel
```
Follow the prompts (no build settings needed — it's a static file). Vercel also serves HTTPS by default.

## 3. Swapping in your own artwork (once you're ready)

You'll need to:
1. **Compile your artwork image into a `.mind` file.** Easiest path right now (no backend needed yet): use MindAR's free browser-based compiler at their official GitHub Pages tool — search "MindAR image target compiler" — upload your artwork photo, download the `.mind` file it generates.
2. Host that `.mind` file and your overlay video somewhere with a public HTTPS URL (a GitHub repo, Vercel static folder, S3 bucket, or Firebase Storage all work).
3. In `index.html`, replace:
   - `imageTargetSrc: https://.../card.mind` → your `.mind` file's URL
   - the `<video src="...">` → your overlay video's URL
4. Update the `<a-video width="1" height="0.552">` ratio to match your artwork image's actual aspect ratio (`height = width × imageHeight / imageWidth`).

## 4. Generating the QR code

Once deployed, generate a QR code that simply encodes your live URL (e.g. `https://you.github.io/repo/`). Any free QR generator works (`qrcode` npm package, or a web tool) — no special AR-aware QR format is needed, since the "AR" happens entirely inside the webpage the QR code opens.

## Known limitations of this prototype (to solve in later steps)
- Only supports **one hardcoded artwork**. Multiple artworks will need a dynamic `/view/:artworkId` route once we build the backend.
- No admin upload UI yet — `.mind` compilation is done manually via MindAR's hosted tool for now.
- iOS Safari requires the overlay video to start **muted** (handled in the code) due to autoplay restrictions; the mute button lets viewers opt in to sound.
