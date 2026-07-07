---
name: browser
description: Drive a real browser via the browser-use CDP CLI (navigate, screenshot, coordinate-click, extract text/forms, manage tabs). Use for end-user QA journeys, form automation, scraping, and any web automation. Requires Chrome remote-debugging enabled; coordinate-clicking needs a vision-capable model or a js-reachable container.
license: MIT
allowed-tools: ["bash", "read", "write", "edit", "describe_image"]
---

# Browser — `browser-use` (CDP harness)

Direct browser control via CDP, attached to your **real Chrome** → existing logins/SSO work.
Backed by the `browser-use` CLI (PyPI package `browser_harness`; docs/self-id call it
`browser-harness`, but the **installed command is `browser-use`**, shorthand `bu`).

## ⚠️ Preconditions — verify before any browser work

```bash
browser-use --doctor
```

Must show:
- `[ok] chrome running`
- `[ok] daemon alive`
- `[ok] active browser connections` — **≥ 1**

### 1. Chrome remote-debugging (HUMAN step, once)

If doctor shows `active browser connections — 0`:

1. In Chrome, open `chrome://inspect/#remote-debugging`
2. Tick **"Allow remote debugging for this browser instance"**
3. Click **Allow** on the popup that appears
4. Re-run `browser-use --doctor` → connections ≥ 1

No `--remote-debugging-port` flag / Chrome restart needed — v3 uses Chrome's built-in toggle.
The "Chrome is being controlled by automated test software" banner is normal — it just means
CDP is attached; leave it (turning it off disables remote-debug).

### 2. Vision-capable model (for the coordinate-click loop)

The core interaction is **screenshot → read pixel → `click_at_xy(x, y)` → screenshot**.
On a text-only model this is crippled — every screenshot needs a separate image-analysis call
(per-click tax) and is imprecise.

- **Click-heavy session → start pi on a vision-capable model** (one that reads images in-context).
- **Text-model fallback:** prefer `js(...)` / DOM extraction (`page_info()`, selectors) over
  visual clicks. `describe_image` is fine for *describing* a page but **too imprecise for click
  coordinates** (validated 2026-07-07: derived coord was 160px off → missed a 162px-tall button).
  Do **not** feed `describe_image`-derived coords into `click_at_xy`. On a text model, use
  `js(...)` selector-clicks (`document.querySelector(...).click()`) instead — OR the
  container-coords technique in **Edge cases** below for js-unreachable targets.

## Canonical reference (READ on first use — do not duplicate here)

```bash
browser-use skill
```

That prints the authoritative helper catalogue (`new_tab`, `ensure_real_tab`, `page_info`,
`capture_screenshot`, `click_at_xy`, `wait_for_load`, `js`, `cdp`, `start_remote_daemon`,
`stop_remote_daemon`, …) + interaction-skills pointers (dialogs, iframes, shadow-dom, tabs…).
The sections below are the **pi-specific delta only**.

## Core loop — local/staging QA

```bash
browser-use <<'PY'
ensure_real_tab()
new_tab("http://localhost:3000")   # first nav is new_tab(), NOT goto_url()
wait_for_load()
print(page_info())
capture_screenshot("/tmp/browsershot.png")
PY
```

- Helpers are pre-imported inside the heredoc; the daemon auto-starts.
- Click = screenshot → derive `(x,y)` → `click_at_xy(x,y)` → screenshot to confirm.
- Analyze the screenshot natively (vision model) or via the `describe_image` tool (text model).
- For DOM-bound work (text extraction, form values) prefer `js(...)` over coordinates.

## Cloud (stealth) browsers — bot-protected sites

Local CDP = YOUR Chrome (fine for localhost, staging, SSO'd sites). For bot-protected targets
(e.g. upwork.com), spin up an isolated cloud daemon:

```bash
browser-use auth login                 # one-time — your Browser Use Cloud account
browser-use <<'PY'
start_remote_daemon("c1")              # short made-up name
PY
BU_NAME=c1 browser-use <<'PY'          # MUST prefix BU_NAME for all calls to this browser
new_tab("https://example.com")
PY
```

- Cloud browsers **bill until stopped** — before ending a task, ask the user "close this browser?"
  and if yes run `stop_remote_daemon("c1")`.
- Cookie/profile sync between local + cloud: see `browser-harness/interaction-skills/profile-sync.md`.

## Safety (policy — enforced in prose)

- **Never reuse the user's visible tab** — always `new_tab(...)`; close what you opened.
- **Login walls:** stop and ask. Exception: if Chrome is already SSO'd, use it — but still
  stop for passwords, MFA, consent prompts, or ambiguous account choice.
- **Destructive actions** (submit/delete/publish): confirm with the user first.
- **Orphan cleanup:** if a session crashes mid-browser, sweep with `pkill -f browser_harness`.
- Don't leave cloud browsers running unattended.

## Edge cases (validated 2026-07-07)

**Closed shadow-DOM & cross-origin iframes — use container coords.**
js selectors cannot reach into closed shadow-DOM (`host.shadowRoot === null`) or cross-origin
iframes (`contentDocument === null`). `click_at_xy` penetrates both at the compositor level.
To aim without vision: get the `getBoundingClientRect()` of the js-reachable **container** (the
shadow host element, or the `<iframe>` element), compute its center, and `click_at_xy(cx, cy)` —
the click routes through the boundary to the centered child. Validated: closed-shadow button +
cross-origin iframe button both clicked successfully this way (text → "… CLICKED ✓").
This means the v3 headline capabilities work on a **text model** — no vision needed when a
js-reachable container exists.

**JS dialogs (alert/confirm/prompt) — neutralize before interacting.**
A `window.alert()` triggered during a `click_at_xy`-driven click is auto-dismissed by the
environment in a way that **aborts the handler's subsequent statements** (the post-`alert()`
lines never run). Neutralize dialogs upfront:
`js("window.alert = window.confirm = window.prompt = () => {}")` — then clicks complete
their handlers normally.

## Known limitation

Pure-visual coordinate derivation (canvas, image maps — targets with **no** js-reachable
container) requires a vision model that reads images in-context. If your pi setup's `read` tool
doesn't pass images to the model, coordinate-clicking such targets is unavailable — use
`js(...)` or the container-coords technique instead. This is a pi image-passing limitation, not
a browser-use limitation.