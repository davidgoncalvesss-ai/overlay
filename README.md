# TikTok Text Overlay — Web version

Single-page web app. Drag videos in, edit the 3 lines, download the result. Everything runs in the browser — no backend, no uploads, no paid infra.

**How it works:** FFmpeg compiled to WebAssembly runs on the user's machine. Text is rendered into a transparent PNG using a `<canvas>` (so emoji render natively through the OS, looking correct everywhere), then overlaid onto the video with FFmpeg's `overlay` filter. Style matches the original AppleScript: Inter SemiBold, font size 64, 3 lines, thick black outline.

---

## Deploy (GitHub → Vercel)

1. Create a new GitHub repo and push these files to it:

   ```bash
   cd tiktok-text-overlay-web
   git init
   git add .
   git commit -m "initial"
   git branch -M main
   git remote add origin https://github.com/YOU/tiktok-text-overlay.git
   git push -u origin main
   ```

2. Go to [vercel.com/new](https://vercel.com/new), sign in with GitHub, and import the repo. Click **Deploy**. Vercel detects it as a static site and uses the headers in `vercel.json` automatically.

3. After ~30 seconds you'll get a URL like `https://tiktok-text-overlay-xxx.vercel.app`. Open it, wait for the "Ready" banner (first visit downloads ~30 MB of FFmpeg WASM, cached afterward), and drop videos.

Every `git push` to `main` redeploys automatically.

---

## Deploy locally for testing

You can't just double-click `index.html` — FFmpeg.wasm needs COOP/COEP headers that `file://` doesn't provide. Run a tiny server:

```bash
npx serve --cors --listen 3000
# or
python3 -m http.server 3000
```

Then open `http://localhost:3000`. On localhost, SharedArrayBuffer works even without the headers.

---

## Tweaking the text style

All style constants live at the top of the `<script type="module">` block in `index.html`:

```js
const FONT_SIZE    = 64;     // px, scales proportionally with video width
const FONT_WEIGHT  = 600;    // Inter SemiBold
const STROKE_WIDTH = 10;     // thickness of black outline
const LINE_SPACING = 1.15;   // line height multiplier
```

The text is centered on the frame by default. To move it, edit `renderOverlayPNG()` — change `startY` to push it up or down:

```js
// Upper-middle (classic TikTok):
const startY = height * 0.25;

// Lower-third:
const startY = height * 0.7;
```

To change the default text, edit the three `<input value="...">` attributes.

---

## Tradeoffs vs. the other versions

| | This (web/WASM) | AppleScript | Cloudflare |
|---|---|---|---|
| Deploy | `git push` | save script | wrangler + Docker |
| Cost | Free | Free | Paid plan + per-minute compute |
| Speed per video | 3–5× slower than native | Native (fastest) | Native on CF |
| First load | ~30 MB download | — | Tiny |
| Works on phone | ✓ | ✗ | ✓ |
| Max file size | ~1–2 GB (browser RAM) | Unlimited | Up to 500 MB/upload |

For typical TikTok clips (30 seconds, 1080p), expect 30–90 seconds of processing per video in the browser. Processing one video at a time is serial by design — FFmpeg.wasm has a single virtual filesystem and can't run parallel jobs cleanly.

---

## Files

```
.
├── index.html      # Everything — HTML, CSS, JS, all inline
├── vercel.json     # COOP/COEP headers for SharedArrayBuffer
├── package.json    # Minimal stub for Vercel detection
└── .gitignore
```

---

## Troubleshooting

**Banner says "Failed to load FFmpeg: SharedArrayBuffer not available"** — you're running it on a host that doesn't set COOP/COEP headers. Vercel with the included `vercel.json` handles this. Netlify needs a `_headers` file. GitHub Pages needs a service-worker workaround (see `coi-serviceworker` on GitHub).

**Emoji renders as a tofu box** — your OS doesn't have a color emoji font installed. Rare in 2026 but it happens on minimal Linux installs. Install `fonts-noto-color-emoji`.

**Text looks too small/big on a particular video** — the overlay scales with video width (base is 1080 px). For edge cases (very narrow or very wide videos), adjust the `scale` clamp in `renderOverlayPNG()`.

**Safari crashes on large videos** — Safari has a stricter ArrayBuffer size limit than Chrome/Firefox. For videos over ~500 MB, use Chrome or Firefox.
