<div align="center">

# TickBox

A to-do list that syncs to your own Puter account — no signup form, no backend, no API keys

[![Live][badge-site]][url-site]
[![HTML5][badge-html]][url-html]
[![CSS3][badge-css]][url-css]
[![JavaScript][badge-js]][url-js]
[![Puter.js][badge-puter]][url-puter]
[![Claude Code][badge-claude]][url-claude]
[![License][badge-license]](LICENSE)

[badge-site]:    https://img.shields.io/badge/live_site-0063e5?style=for-the-badge&logo=googlechrome&logoColor=white
[badge-html]:    https://img.shields.io/badge/HTML5-E34F26?style=for-the-badge&logo=html5&logoColor=white
[badge-css]:     https://img.shields.io/badge/CSS3-1572B6?style=for-the-badge&logo=css3&logoColor=white
[badge-js]:      https://img.shields.io/badge/JavaScript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black
[badge-puter]:   https://img.shields.io/badge/Puter.js-181818?style=for-the-badge&logoColor=white
[badge-claude]:  https://img.shields.io/badge/Claude_Code-CC785C?style=for-the-badge&logo=anthropic&logoColor=white
[badge-license]: https://img.shields.io/badge/license-MIT-404040?style=for-the-badge

[url-site]:   https://tickbox.neorgon.com/
[url-html]:   #
[url-css]:    #
[url-js]:     #
[url-puter]:  https://developer.puter.com
[url-claude]: https://claude.ai/code

</div>

---

## Overview

TickBox is a to-do list whose tasks follow you between browsers without you setting up an account with *this* site. Sign in with Puter and the list lives in your own cloud storage; skip signing in and it works exactly the same, just confined to the browser you are using.

There is no server here to trust. The site is static files, and the only thing standing between the list and the cloud is your own Puter account — which means there is also no API key to leak, no database to run, and no per-user cost to the site owner.

**Live:** tickbox.neorgon.com

---

## Features

- **Works before you sign in** -- The list is fully usable signed out, backed by `localStorage`. Signing in later adopts what you already added instead of discarding it.
- **Cross-browser sync** -- Signed in, the list is one entry in your Puter key-value store, so the same tasks appear on any browser you sign in from.
- **Honest sync status** -- A header badge reads Local, Syncing, Synced, or Sync failed. A failed cloud write says so rather than pretending the task was saved everywhere.
- **Conflict resolution that respects deletes** -- Two devices editing the same list merge per task by last-write-wins, and deletes are tombstoned so a delete on one device is not resurrected by a stale copy on another.
- **Inline editing** -- Enter saves, Escape discards, clicking away saves. No modal for a one-line change.
- **Filters and bulk clear** -- All / Open / Done, plus a Clear done that sweeps finished tasks in one write.

---

## How sync works

The list is stored as a **single** key-value entry (`tickbox:tasks`) rather than one entry per task. A few thousand tasks sit far under Puter's 400 KB value ceiling, and one write per action keeps the list internally consistent — there is no half-applied batch to recover from.

Each task carries `updatedAt`, and deletes set `deletedAt` instead of removing the record. On load, the cloud copy and the local copy are merged by id, keeping whichever version of a task was written most recently. Because a delete is a record rather than an absence, it wins over an older edit made elsewhere — the common failure mode of naive sync, where a task you deleted on your laptop reappears from your phone. Tombstones older than 30 days are pruned.

Writes always hit `localStorage` first and synchronously, so a refresh never loses work even if the network call fails.

---

## Running locally

ES modules and Puter.js both require an HTTP server (not `file://`):

```bash
make serve      # → http://localhost:8859
```

No build step, no `npm install`, no API keys.

---

## Architecture

```
tickbox-site/
├── index.html          # Shell: header, auth sheet, compose form, list
├── css/
│   └── style.css       # Tokens + app styles (header/footer kits vendored alongside)
└── js/
    ├── app.js          # Entry point — paints local list, then reconciles with cloud
    ├── store.js        # Storage layer: localStorage ↔ puter.kv, merge + tombstones
    ├── state.js        # Shared state and task mutations
    ├── render.js       # Rebuilds list, summary, filters, sync badge from state
    ├── events.js       # Delegated listeners; owns the auth flow
    └── utils.js        # escHtml, toast, $ helper
```

**Data flow:** every user action mutates `state`, calls `render()` immediately, then persists — so the UI never waits on the network, and the sync badge reflects what actually happened.

**Puter APIs used:** `puter.auth.signIn/signOut/isSignedIn/getUser` for identity, `puter.kv.get/set` for storage.

> **Gotcha:** any `puter.kv` call auto-triggers authentication, which opens a popup. Browsers block popups outside a user gesture, so every cloud call in `store.js` is gated behind `isSignedIn()` — otherwise a page load while signed out fires a popup the browser kills.

---

<div align="center">
<sub>Part of <a href="https://neorgon.com/">Neorgon</a> · <a href="https://developer.puter.com">Powered by Puter</a></sub>
</div>
