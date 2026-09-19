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

## Counting sessions from the server access log

The app stays a single static file — there's no beacon, no backend, and no
IP tracking built into the page. Every session still gets a random id that's
kept in the URL as `?s=<id>`, and the app forces one real page load with that
`?s=` id the very first time a visitor arrives (not just a client-side
`history.replaceState`), so the id always shows up in your web server's
normal access log — including first-time visitors, not just reloads.

That means you can count distinct sessions with the access log you already
have, with no extra instrumentation:

```bash
# Total distinct sessions ever seen
grep -oP 'GET /bingo\.html\?s=\K[a-zA-Z0-9]+' /var/log/apache2/bingo-access.log \
  | sort -u | wc -l

# Distinct sessions per day (Apache "combined" log format)
awk -F'"' '
  $2 ~ /GET \/bingo\.html\?s=/ {
    match($1, /\[[0-9]{2}\/[A-Za-z]{3}\/[0-9]{4}/); day = substr($1, RSTART+1, RLENGTH-1);
    match($2, /s=[a-zA-Z0-9]+/); sid = substr($2, RSTART+2, RLENGTH-2);
    print day, sid;
  }' /var/log/apache2/bingo-access.log | sort -u | cut -d' ' -f1 | uniq -c
```

(Adjust the log path and field numbers if your `LogFormat` differs from the
Apache `combined` default.)

Notes:

- This only counts *sessions* (i.e., games started/resumed), not IP
  addresses — nothing in the app or these commands looks at the visitor's
  IP. Apache logs the client IP anyway as part of normal operation (and you
  can make it the real visitor IP behind Cloudflare Tunnel with
  `mod_remoteip` + `RemoteIPHeader CF-Connecting-IP`), but that's independent
  of, and not required for, session counting.
- Because the page never makes any further network requests after it loads
  (all card/goal/call interactions are pure client-side JS), the access log
  can tell you *how many* sessions happened and *when* they started, but not
  how long someone actually spent playing. Getting real dwell/interaction
  time would require adding a small JS beacon and a backend endpoint, which
  this project intentionally does not have.

## Project files

- `bingo.html` — the entire application (markup, styles and script).
- `manual.html` — the bilingual user manual (EN/ES in one static page,
  language chosen via `?lang=en|es`, with its own EN/ES switch).
- `README.md` — this file.

Editor/IDE folders (e.g. `.vscode/`, `.vs/`) are local tooling metadata and
are intentionally not part of the published app.