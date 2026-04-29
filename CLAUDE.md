# Brand to Scale — Podcast Landing Page

**Domain:** brandtoscale.co.uk  
**Deployment:** GitHub → Vercel (static, no build step)  
**Stack:** Single-file `index.html` — vanilla HTML, CSS, JS. No framework, no bundler, no dependencies.

---

## File Structure

```
index.html   — entire site (HTML + CSS + JS, self-contained)
CLAUDE.md    — this file
```

---

## Brand

### Colours
| Name          | Hex       | Usage                              |
|---------------|-----------|------------------------------------|
| Dawn          | `#191919` | Primary background                 |
| Dragon Fire   | `#FE6D4C` | Accent, CTAs, highlights           |
| Dusk          | `#FAF8F7` | Light text, light section text     |
| Pixie Pink    | `#FE86F6` | Secondary accent (available)       |
| Curious Blue  | `#3052F9` | Secondary accent (available)       |
| Mindaro       | `#C4FE79` | Secondary accent (available)       |
| Space Green   | `#00524D` | Secondary accent (available)       |
| Ultra Violet  | `#9A4EFF` | Secondary accent (available)       |

### Typography
- **Primary:** Satoshi (loaded from Fontshare CDN)
  - `https://api.fontshare.com/v2/css?f[]=satoshi@300,400,500,700,900&display=swap`
  - Headlines: Satoshi 900 (Black), `line-height: 0.95–1.1`, `letter-spacing: -0.02 to -0.035em`
  - CTAs: Satoshi 700, uppercase, `letter-spacing: 0.07em`
- **Secondary:** P22 Mackinac Pro → fallback `Georgia, 'Times New Roman', serif`
  - Used for subheadings, episode descriptions, body serif text

### Aesthetic
Dark editorial. Bold headlines on `#191919`. Dragon Fire orange for every CTA and accent. Confident, not loud. Generous whitespace. Sentence case everywhere except button labels (uppercase).

---

## Data Sources

### Transistor RSS Feed
```
https://feeds.transistor.fm/brand-to-scale
```
- Loaded client-side via CORS proxy: `https://api.allorigins.win/get?url=ENCODED_URL`
- Parsed as XML using `DOMParser`
- Namespace for iTunes tags: `http://www.itunes.com/dtds/podcast-1.0.dtd`
- Episode ID extracted from GUID: `https://share.transistor.fm/s/EPISODE_ID`

**Transistor player embed format:**
```
https://player.transistor.fm/episodes/EPISODE_ID?color=FE6D4C
```
(colour is hex without `#` prefix)

### YouTube Channel
```
https://www.youtube.com/@alchemybrandingstudio/podcasts
```
Latest video is loaded dynamically:
1. Fetch channel page via allorigins proxy → extract `channelId` (`UC...`)
2. Fetch YouTube RSS: `https://www.youtube.com/feeds/videos.xml?channel_id=CHANNEL_ID`
3. Parse XML, extract `<yt:videoId>` (namespace: `http://www.youtube.com/xml/schemas/2015`)
4. Embed as `https://www.youtube.com/embed/VIDEO_ID`
5. Fallback: styled click-through card to channel if any step fails

---

## Page Sections (in order)

1. **Fixed Nav** — logo left, links right (Latest / Watch / All Episodes), Spotify Follow CTA
2. **Hero** — full-viewport, Satoshi 900 headline, serif subheading, CTA buttons, platform links
3. **Latest Episode** (`#latest`) — auto-loaded from RSS: artwork, title, meta, description, Transistor player iframe, Spotify follow
4. **YouTube** (`#watch`) — latest video embed (dynamic) + Subscribe CTA
5. **All Episodes** (`#episodes`) — full list from RSS (items[1..n]), episode number, title, date, duration, description snippet, link to Spotify/Transistor
6. **About Strip** — Dragon Fire background, show description, link to `alchemybranding.studio`
7. **Footer** — logo, platform links, copyright year (auto)

---

## JavaScript Architecture

All JS is in a single `<script>` block at the bottom of `index.html`.

### CONFIG object
```js
const CONFIG = {
  rssUrl:     'https://feeds.transistor.fm/brand-to-scale',
  proxyBase:  'https://api.allorigins.win/get?url=',
  spotifyUrl: '',   // ← UPDATE with actual Spotify show URL
  appleUrl:   '',   // ← UPDATE with actual Apple Podcasts show URL
  ytChannel:  'https://www.youtube.com/@alchemybrandingstudio/podcasts',
};
```

### Key functions
| Function | Purpose |
|---|---|
| `loadRSS()` | Fetches and parses Transistor RSS feed |
| `renderLatest(item, fallbackArt)` | Populates the featured episode section |
| `renderList(items, fallbackArt)` | Renders the full episode list |
| `tryExtractPlatformUrls(xml)` | Scans RSS feed for Spotify/Apple URLs, updates all `[data-platform]` links |
| `loadYouTube()` | Fetches channel ID → YT RSS → embeds latest video |
| `updatePlatformLinks(platform, url)` | Updates all elements with `data-platform="spotify|apple"` |
| `extractEpisodeId(url)` | Extracts Transistor episode ID from share URL |
| `getItunesText(el, tag)` | Namespace-safe getter for `itunes:*` text content |
| `getItunesAttr(el, tag, attr)` | Namespace-safe getter for `itunes:*` attributes |
| `formatDuration(str)` | Handles `HH:MM:SS`, `MM:SS`, and raw-seconds formats |

### Platform link pattern
All Spotify/Apple links in the HTML carry `data-platform="spotify"` or `data-platform="apple"`. When `CONFIG.spotifyUrl` / `CONFIG.appleUrl` are set (or detected from RSS), `updatePlatformLinks()` updates all of them at once.

### Animations
`IntersectionObserver` drives `.fade-in` → `.fade-in.visible` transitions (opacity + translateY). Hero elements fire immediately on load. Respects `prefers-reduced-motion`.

---

## What Needs Manual Updating

| Item | Where | Action |
|---|---|---|
| Spotify show URL | `CONFIG.spotifyUrl` in `<script>` | Add `https://open.spotify.com/show/XXXXXXXXXX` |
| Apple Podcasts show URL | `CONFIG.appleUrl` in `<script>` | Add `https://podcasts.apple.com/...` |
| YouTube playlist ID | `CONFIG.ytPlaylistId` in `<script>` | Set to `PLYxVX2SN4SFfN_SNkRRCYJo1T6hYH82JI` ✓ — playlist embed always starts at video 1, no API calls |
| OG image | `<head>` | Add `<meta property="og:image" content="...">` once artwork is available |
| Favicon | `<head>` | Add `<link rel="icon" ...>` |

---

## Deployment

1. Push all files (`index.html`, `CLAUDE.md`, `vercel.json`, `netlify.toml`) to GitHub repo
2. Connect repo to **Vercel** or **Netlify** — both are pre-configured via `vercel.json` / `netlify.toml`
   - Both configs set up same-origin proxy rewrites for `/_rss` → Transistor RSS feed and `/_yt` → YouTube channel page (eliminates CORS issues without relying on third-party proxies)
3. Set custom domain to `brandtoscale.co.uk` in project settings

---

## Produced by
[Alchemy Branding Studio](https://alchemybranding.studio)
