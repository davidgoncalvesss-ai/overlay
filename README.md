# Bulk TikTok Text Overlay

Upload one video, paste a list of captions, get back a batch of videos — each with a different caption rendered in TikTok-style text. Runs 100% in the browser. No server, no uploads.

## What it does

- Drop a vertical (or any) video in
- Paste captions, one per line
- Pick a style (Classic / Black bar / White bg / Outline / Neon)
- Adjust font size, position, colors, weight
- Click **Process all** — get one video per caption, downloadable as WebM or MP4

MP4 uses `ffmpeg.wasm` in the browser (loaded on demand, ~30 MB the first time, then cached). WebM skips the conversion step — instant, but you'll need to re-encode before uploading to TikTok.

## Deploy to Vercel

### Option A — the one-click way (from GitHub)

1. Create a new GitHub repo and push these three files to the root:
   - `index.html`
   - `vercel.json`
   - `README.md`
2. Go to [vercel.com/new](https://vercel.com/new), import the repo
3. Framework preset: **Other** (it's a static site)
4. Click **Deploy**

That's it. Vercel serves `index.html` at your domain. The `vercel.json` adds the COOP/COEP headers that `ffmpeg.wasm` needs for MP4 encoding.

### Option B — Vercel CLI

```bash
npm i -g vercel
cd tiktok-overlay
vercel --prod
```

### Option C — local testing

Open `index.html` in Chrome directly **and it mostly works** — but MP4 conversion will fail because `file://` can't use `SharedArrayBuffer`. Run a local server with the right headers:

```bash
npx http-server -p 8080 --cors \
  -H "Cross-Origin-Opener-Policy: same-origin" \
  -H "Cross-Origin-Embedder-Policy: credentialless"
```

Or simpler, just deploy to Vercel and use that.

## Browser support

- **Chrome / Edge:** works fully, including MP4 export
- **Firefox:** WebM export works. MP4 may or may not depending on version — Firefox's support for `credentialless` COEP is partial
- **Safari:** WebM recording is limited; MP4 export unreliable. Use Chrome

This is a Chromium-first tool. TikTok creators are usually on desktop Chrome anyway.

## How the rendering works

1. For each caption, a fresh `<video>` element plays the source file silently through a `Web Audio` graph (so the audio goes into the recording but not out the speakers)
2. A `<canvas>` redraws each video frame + the caption text via `requestAnimationFrame`
3. `canvas.captureStream()` + the audio track feed a `MediaRecorder` which produces a WebM blob
4. If MP4 is selected, the WebM blob is piped through `ffmpeg.wasm` to re-encode as H.264/AAC MP4 with `+faststart`

## Customizing

All the rendering logic is in the `<script>` at the bottom of `index.html`. Key functions:

- `drawText(ctx, canvas, text, style)` — the text-drawing logic. Change this to add new styles (shadows, gradients, emoji handling, etc.)
- `presets` — add a preset by dropping an entry here and a matching `<button data-preset="...">` in the markup
- `renderOne(text, style, onProgress)` — the per-clip render pipeline

To use a custom font, add the `<link>` to Google Fonts at the top and change the `ctx.font = ...` line in `drawText`.

## Notes

- Output resolution matches the input video exactly. Upload a 1080×1920 vertical video, get 1080×1920 back
- Bitrate is fixed at 8 Mbps which is plenty for TikTok. Change `videoBitsPerSecond` in `renderOne` if you want different
- Safety timeout is 8 seconds past the video duration — if a clip hangs longer than that, recording force-stops
- The tool doesn't store anything. Refresh the page and results are gone

## License

Yours to modify. Build whatever you want with it.
