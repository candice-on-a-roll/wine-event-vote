# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A static, no-build web app for running live voting at champagne/wine tasting events. Four HTML files served directly from the filesystem or any static host (GitHub Pages, etc.). No package manager, no bundler, no server.

## Architecture

All logic lives inline in each HTML file — CSS in `<style>` tags, JS in `<script type="module">` tags. There is no shared JS or CSS across files; each page is self-contained.

**Firebase Realtime Database** (project: `wine-event-vote`, region: `asia-southeast1`) is the only backend. All reads/writes happen directly from the browser using the Firebase JS SDK loaded from CDN (`https://www.gstatic.com/firebasejs/10.12.0/`).

### Data model (`sessions/{sessionId}`)
```
{
  name: string,
  date: string,         // YYYY-MM-DD
  votingOpen: boolean,
  responses: number,    // total vote submissions
  questions: [{ text, type, options? }],   // type: single | multi | rating | text
  votes: {
    q0: { [encodeURIComponent(option)]: count },
    q1: ...
  }
}
```

Vote keys are `encodeURIComponent`-encoded to be valid Firebase paths. Rating answers are encoded as `"1 star"`, `"2 stars"`, etc.

### Pages
| File | Who uses it | Purpose |
|------|-------------|---------|
| `index.html` | Guests | Landing — finds the most recent open session and redirects to `vote.html?session=<id>` |
| `vote.html` | Guests | Multi-step voting wizard; submits via `runTransaction` to avoid race conditions |
| `setup.html` | Host (password-gated) | Create/edit sessions, manage questions, open/close voting |
| `results.html` | Host on a projector | Live results display, navigable per question; supports `?session=<id>&q=<index>` to pin a slide |

### Admin password
Hard-coded in `setup.html` as `const PW = '12345678'`. This is intentional — it's a low-stakes personal event tool.

## Development

Open any HTML file directly in a browser — no build step needed. For live-reload during editing, use any static file server:

```bash
# Python
python -m http.server 8080

# Node (if npx available)
npx serve .
```

Firebase config (API key, project ID, etc.) is embedded in every file and is safe to be public — Firebase security rules govern access.
