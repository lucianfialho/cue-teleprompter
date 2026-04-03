# EatText

> Pac-Man eats the news — one line at a time.

EatText is a PWA e-reader where Pac-Man scrolls through RSS articles. No feeds, no lists, no doom scrolling. Just open it and start eating.

**Live:** [eatext.vercel.app](https://eattext.vercel.app)

---

## What it does

- Fetches articles from BBC Technology, Ars Technica, The Guardian, and The Verge
- Shows the article's hero image above the reading line
- Pac-Man eats the text character by character, left to right
- Arcade scoreboard tracks words eaten and hi-score
- Yolo mode: auto-advances through articles non-stop
- Auto-refreshes when you reach the last article — never runs out

## Controls

| Input | Action |
|---|---|
| Arrow Up / Down | Speed +/– |
| Mouse wheel | Speed +/– |
| ⚙️ button | Settings |

## Settings

- **Speed** 1–10
- **Font size** 24–96px
- **Theme** Dark / Light / Auto
- **Scroll effect** Linear / Fisheye
- **Mirror mode**
- **Progress indicator**
- **Yolo mode** — infinite auto-advance

## Stack

- Vanilla JS, no framework, no build step
- Canvas API for rendering
- Vercel serverless function (`/api/rss`) as RSS proxy — no CORS issues
- Service Worker for offline support
- PWA installable on iOS and Android

## Run locally

```bash
npm i -g vercel
vercel dev
```

Requires Vercel CLI so the `/api/rss` proxy works.
