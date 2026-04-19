# Bulk Video Text Overlay

Upload multiple videos, give each one a caption, render them all as TikTok-ready MP4s — batch download as a ZIP. Runs 100% in the browser. No server, no uploads, no API keys.

## What it does

- Drop in any number of videos (vertical or otherwise)
- Type captions into a single textarea, one per line (line 1 goes on video 1, line 2 on video 2, etc.)
- Pick from 12 font styles, 5 presets, and controls for size, weight, letter spacing, line height, position, colors, stroke, background, case
- Drag the text anywhere on the live preview
- Click Render — get native MP4s back
- Click Download all as ZIP for batch download

## Output quality

MP4 encoded natively via the browser's WebCodecs API (hardware-accelerated H.264). Bitrate scales with resolution: 720p at 10 Mbps, 1080p at 20 Mbps, 1440p at 28 Mbps, 4K at 40 Mbps. Audio is AAC at 192 kbps, 48 kHz stereo. Output has `faststart` so it's upload-ready for TikTok, Instagram, YouTube Shorts.

## Deploy to Vercel

1. Push `index.html` (and optionally `vercel.json`, `README.md`) to a GitHub repo
2. Go to vercel.com/new and import the repo
3. Framework preset: Other
4. Click Deploy

No build step. The `vercel.json` is optional — just sets clean URLs.

CLI alternative:

```bash
npm i -g vercel
cd tiktok-overlay
vercel --prod
```

Or just open `index.html` directly from your filesystem — everything works from `file://` because there's no server dependency.

## Browser support

- Chrome / Edge 94+: full MP4 export with audio (recommended)
- Firefox 130+: MP4 export; audio support depends on `MediaStreamTrackProcessor`
- Safari 16.4+: MP4 export works; audio capture is browser-dependent
- Older browsers: automatic fallback to WebM

## Customizing

Everything is in the `<script>` block at the bottom of `index.html`. Key functions:

- `drawText(ctx, canvas, text, style)` — caption rendering. Add shadows, gradients, emoji handling, etc.
- `presets` — add new style presets (drop an entry plus a matching `<button data-preset="...">`)
- `pickMp4Settings(W, H)` — change bitrate tiers
- `renderVideoToMp4(v, s, onProgress)` — the whole encode pipeline

## Notes

- Output resolution matches input exactly. 1080x1920 in, 1080x1920 out
- H.264 requires even dimensions; odd widths/heights are rounded down by 1 px
- Keyframes every 2 seconds
- Files stay in browser memory until the tab closes — refresh and everything's gone
- For batches of 50+ HD clips, individual downloads are safer than ZIP (avoids memory spikes)

## License

Yours. Do whatever.
