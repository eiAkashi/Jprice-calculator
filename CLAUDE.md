# CLAUDE.md — Jewellery Price Calculator

Project context for Claude Code. This app calculates a Japan-market jewellery
purchase: **price tag → 割 discount → consumption tax → buying-agent fee → final
amount in JPY and MMK (lakh)**. It is used by a buying agent in Tokyo who quotes a
Myanmar-side client, so both currencies matter.

## TL;DR for the agent
- **Two files matter:** `index.html` (the app) and `config.json` (the default MMK rate).
- No framework, no build step, no backend. Pure HTML + CSS + vanilla JS.
- **No `localStorage` / `sessionStorage`.** Persistence is via an exported /
  imported text file (JSON inside a `.txt`).
- Fonts load from Google Fonts CDN with system fallbacks; the app must keep
  working offline (degraded fonts are acceptable).

## File structure
```
index.html     — the full app (HTML + CSS + JS, self-contained except for config)
config.json    — { "mmkRate": 3700 } — edit this to change the default FX rate
.gitignore     — macOS + Python ignores
README.md      — user-facing docs
CLAUDE.md      — this file
```

## Run / preview
```
python3 -m http.server 8000   # then open http://localhost:8000
```
No dependencies, no install. Must be served (not opened as file://) because
`config.json` is fetched via `fetch('config.json')` on startup.

## Config / rate updates
`config.json` is loaded once at startup via `fetch`. If it fails or is absent,
the app falls back to the hardcoded default (3700). To change the default rate:
1. Edit `config.json` — `{ "mmkRate": <new_value> }`
2. Commit and push — GitHub Pages will serve the update within ~1 minute.
The user can also override the rate interactively in the gear panel; that overrides
only their session (or saved settings file), not `config.json`.

## Calculation logic — DO NOT change without owner sign-off
Inputs: `price` (¥ tag), `wari` (折, 0–10, default 4), `taxP` (%, default 10),
`feeP` (%, default 5), `rate` (¥ per 1 lakh MMK, loaded from config.json default 3700).

```
payRate  = 1 - wari/10              # 4折 → pay 0.4 (i.e. 60% OFF)
subtotal = round(price * payRate)   # 割引後価格 (pre-tax)
discount = price - subtotal
tax      = round(subtotal * taxP/100)    # tax is on the discounted price
afterTax = subtotal + tax                # 小計 — what you pay the shop
fee      = round(afterTax * feeP/100)   # fee is on the shop total (afterTax)
total    = afterTax + fee               # = お支払い額 (JPY, the headline figure)
mmkLakh  = total / rate                 # displayed as X.XX lakh
```
Rules:
- Every yen line is rounded to a whole yen so the receipt lines sum exactly to `total`.
- MMK is shown as **lakh with 2 decimals**. Because 1 lakh = ¥`rate`, `lakh = yen / rate`.
- "Effective discount off tag" note = `round((1 - total/price) * 100)`.

Receipt display order: 値札価格 → 割引 → 消費税 → **小計** (`afterTax`, shop total) → 代行手数料 → お支払い額

## Features
- **Discount chips:** quick-select buttons `3折 · 3.5折 · 4折 · 4.5折 · 5折` (default input
  value: 4). A hint below shows e.g. `4折 (割引 −60%)`. The active chip is highlighted.
- **Settings (gear icon, top-right):** collapsible panel holding tax %, fee %,
  and FX rate. Kept out of the main view so the price + final figures get the room.
- **Remember (text file):** *Save* uses the Web Share API on iOS (share sheet) and
  falls back to a blob download on other browsers. Downloads `jewellery-settings.txt`
  — pretty JSON `{rate, tax, fee, rateSetDate, savedAt}`. *Load* reads it back.
- **Rate set-date:** stamped to today only when the rate value actually changes;
  on Load the saved date is restored, not overwritten. Shown in settings and as a
  small note under the total (`レート ¥3,700/lakh · 設定 YYYY-MM-DD`).

## Browser compatibility
- Flexbox `gap` has `> * + *` margin fallbacks for Safari < 14.1.
- Save uses `navigator.canShare({files})` → Web Share API on iOS Safari 15+,
  falls back to `<a download>` blob on Chrome / desktop Safari.
- `AbortError` (user cancels share sheet) is swallowed silently.

## Key DOM ids (all wired through one `calc()`)
`price, wari, tax, fee, rate, rateDate, saveBtn, loadBtn, loadFile, status,
gearBtn, settings, rTotal, rMmk, rPrice, rDisc, rSub, rTax, rFee, rSaved, rateNote`.
`animateTotal()` runs a count-up that drives both the JPY headline and the MMK line.

## Design tokens
- Palette: ink `#16110C`, surface `#211912`, raised `#2B2118`, gold `#C9A268`,
  gold-bright `#E8CE9A`, gold-deep `#8A6E42`, cream `#F4ECDD`, muted `#A6967D`,
  negative `#C98A6A`.
- Type: display/serif **Cormorant Garamond** (title + headline number), body/UI
  **Inter** (tabular figures for all amounts).
- Signature: faceted-diamond mark; itemized "appraisal slip" receipt; headline
  total in large gold serif.

## PWA
- Embedded **inline** (no external asset files): web app manifest (base64 data
  URI), `apple-touch-icon` (base64 PNG — the gold diamond), and an SVG favicon.
- iOS: `apple-mobile-web-app-capable`, `black-translucent` status bar,
  safe-area insets via `viewport-fit=cover` + `env(safe-area-inset-*)`.
- Regenerate the icon with `tools/build_icon.py` (writes `icon512.b64`), then
  re-embed with `tools/inject.py`. NOTE: `inject.py` is idempotent — it skips if
  an `apple-touch-icon` link already exists. To re-inject after changing the icon,
  delete the existing `<link rel="apple-touch-icon" ...>` line first.

## Constraints / decisions (already made)
- No localStorage. No build step. Fonts via CDN with fallback.
- `config.json` fetch is fire-and-forget: failure → silent fallback to default rate.
- Numbers must never become iOS tap-to-call (`format-detection: telephone=no`).
- Reduced motion disables the entrance + count-up animations.

## Backlog (suggested order)
1. **Offline service worker (`sw.js`).** Cache `index.html`, `config.json`, and the
   Google Font CSS/woff2 so it runs with zero network.
2. **Self-host / inline fonts** for correct typography offline (or cache via SW).
3. **Optional features:** reverse mode (enter MMK budget → max ¥ price tag at a
   chosen 割); multi-item tally; USD line; a saved-quotes history.
