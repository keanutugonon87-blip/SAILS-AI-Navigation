# ⚓ Bridge Watch — BSMT Study Console

A self-contained study companion for BSMT (Bachelor of Science in Marine
Transportation) freshmen — COLREGS, navigation, seamanship, maritime law,
PEME/PFT prep, and more. Single-page app, no build step, deployable straight
to GitHub Pages.

## Features

- **Ship's Log** — chat-style Q&A that's source-checked against IMO, PCG,
  MARINA, DOH, and official COLREGS/SOLAS/MARPOL/STCW texts before answering.
- **Course Map** — the CHED → MHEI → MARINA → OICNW pathway, with a searchable
  glossary.
- **Quiz** — topic-picker multiple choice with live scoring.
- **Flashcards** — a core COLREGS deck plus AI-generated, source-verified
  decks per topic.
- **Milestones** — tracks PEME, PFT, BT, SDSD, D-WATCH, OBT, and the MARINA
  licensing exam.

## How it's built

One file, `index.html` — HTML, CSS, and vanilla JS, no framework, no build
tools. Progress (quiz stats, flashcard mastery, milestones, chat history) is
saved in the browser via `localStorage`, so it's private to each visitor's
device and doesn't need a database.

## ⚠️ Before this works: set up the AI backend

The AI-powered parts (Ship's Log answers, quiz generation, flashcard
generation) call the Anthropic API. A browser page can never hold an API key
safely — anyone can read it via view-source — so `index.html` instead calls
**your own backend proxy**, which holds the key server-side.

This repo ships with a ready-made proxy for **Google Apps Script**
(`backend/apps-script-proxy.gs`), matching the stack you're already using for
GIC. To wire it up:

1. Open `backend/apps-script-proxy.gs` and follow the setup comments at the
   top — create an Apps Script project, paste the code in, add your
   `ANTHROPIC_API_KEY` as a Script Property (never hardcode it), and deploy
   as a Web App.
2. Copy the deployed Web App URL (ends in `/exec`).
3. Open `index.html`, find this near the top of the `<script>` block:
   ```js
   const BACKEND_ENDPOINT = "PASTE_YOUR_BACKEND_PROXY_URL_HERE";
   ```
   and paste your URL in.
4. Commit and push. Until step 3 is done, the AI features will show a
   graceful "Signal lost" message in the log instead of failing silently —
   the rest of the app (Course Map, Milestones) works either way.

Prefer a different backend (Cloudflare Worker, small Node/Express server,
etc.) instead of Apps Script? Any proxy that accepts a POST with a JSON body
and forwards it to `https://api.anthropic.com/v1/messages` with your API key
attached will work — just point `BACKEND_ENDPOINT` at it. Note the client
sends the request with `Content-Type: text/plain` on purpose, to dodge CORS
preflight (Apps Script can't handle `OPTIONS` requests); your proxy should
still `JSON.parse()` the body as JSON.

## Deploying to GitHub Pages

1. Push this repo to GitHub.
2. Repo → Settings → Pages → Source: deploy from branch → `main` → `/ (root)`.
3. Your site will be live at `https://<your-username>.github.io/<repo-name>/`.

## File structure

```
├── index.html                    # the whole app
├── backend/
│   └── apps-script-proxy.gs      # Google Apps Script backend (copy into script.google.com)
├── .gitignore
└── README.md
```

## Notes

- No API keys, tokens, or secrets are committed to this repo — the
  `.gitignore` also guards against accidentally adding any.
- Data saved by the app (quiz scores, mastered flashcards, milestones, chat
  history) lives only in each visitor's browser (`localStorage`). Clearing
  browser data resets it. There's no shared/multi-device sync built in.
