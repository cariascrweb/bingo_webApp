# Bingo Tracker

A single-page, no-build, no-backend Bingo companion app. Open `bingo.html` in a
browser and use it to track a 75-ball Bingo game: mark called numbers, fill in
your own cards, define custom winning patterns (goals) and get alerted the
moment one is completed — all in English or Spanish.

## Features

- **Live caller board** — tap or type any number 1–75 to mark it called, see
  the last ball, call count, numbers left, and full call history.
- **Dynamic cards** — add or remove as many personal cards as you want
  (`+ Add card` / `Remove card`), no fixed limit.
- **Column validation** — each cell only accepts numbers from its own column
  range (B 1–15, I 16–30, N 31–45, G 46–60, O 61–75); invalid or duplicate
  numbers are rejected with inline feedback.
- **Custom goals** — mark cells on a blank card (or use the Four corners /
  Letter X / Blackout presets) to define a winning pattern, save as many goals
  as you like, and get an on-screen alert the instant any of your cards
  completes one.
- **English / Spanish** — a language switch translates every label, hint and
  message.
- **Built-in user manual** — a **User manual / Manual de usuario** link in the
  top bar opens `manual.html` in a new tab, already in the language currently
  selected in the app (and carrying the session id, so the manual's back link
  returns to the same game).
- **No login, per-session storage** — each browser tab gets its own session
  id (carried in the URL as `?s=...`). All data (called numbers, cards,
  goals, language) is saved in that browser's `localStorage` under the
  session id, so it persists across reloads without any account or server
  database. Use **Copy link** to bookmark/share a session so you can come
  back to the exact same game later, or **New session** to start a fresh,
  empty one in the current tab.

## Running it locally

No build step, no dependencies. Just open the file directly:

```powershell
start bingo.html
```

Or serve it with any static file server, for example:

```powershell
python -m http.server 8080
```

then browse to `http://localhost:8080/bingo.html`.

Keep `manual.html` next to `bingo.html` — the manual link is a relative link
to `manual.html` in the same folder.

## Publishing / self-hosting

`bingo.html` is fully static (HTML/CSS/JS in one file, no server code, no
database). To publish it from a home server:

1. Serve the folder with any static web server (nginx, Caddy, `python -m
   http.server`, `npx serve`, etc.).
2. Expose it to the internet with a [Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/)
   pointing at that local server.

Because all game state lives in each visitor's own browser (`localStorage`,
keyed per tab/session), multiple people can use the same public URL at the
same time without a login and without their data mixing — each tab/device
keeps its own private session automatically.

## Project files

- `bingo.html` — the entire application (markup, styles and script).
- `manual.html` — the bilingual user manual (EN/ES in one static page,
  language chosen via `?lang=en|es`, with its own EN/ES switch).
- `README.md` — this file.

Editor/IDE folders (e.g. `.vscode/`, `.vs/`) are local tooling metadata and
are intentionally not part of the published app.