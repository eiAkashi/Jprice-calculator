# 宝飾計算 · Jewellery Price Calculator

A lightweight, installable web app that turns a Japanese price tag into the final
amount you pay — applying the 割 (wari) discount, consumption tax, and a buying-agent
handling fee — and shows the result in both **JPY** and **MMK (lakh)**.

## Files

| File | Purpose |
|------|---------|
| `index.html` | The app — HTML + CSS + vanilla JS, no framework, no build step |
| `config.json` | Default MMK exchange rate — edit this to update the rate |

## What it does

1. Enter the **price tag** (¥) and the **discount** in 折 using the quick chips
   (3折 · 3.5折 · 4折 · 4.5折 · 5折, default 4折) or type any value.
   A hint below the input shows e.g. `4折 (割引 −60%)`.
2. Behind the **gear icon**: adjust consumption tax (default 10%), handling fee (default 5%),
   and the ¥/lakh FX rate.
3. Read the itemized receipt — **Price tag → Discount → 割引後金額 (amount due) → Tax → 小計 (shop total) → Fee** —
   and the headline **You pay** total in ¥ and lakh MMK.
4. A small note under the total shows the **effective discount off tag** (rounded %) and the
   active rate with its set date.

The math (all yen rounded per line so the receipt sums exactly):

```
payRate  = 1 − wari/10                    # 4折 → pay 0.4 (60% off)
subtotal = round(price × payRate)         # discounted price (pre-tax)
tax      = round(subtotal × tax%)         # tax on discounted price
afterTax = subtotal + tax                 # 小計 — what you pay the shop
fee      = round(afterTax × fee%)         # agent fee on shop total
total    = afterTax + fee                 # お支払い額 (the headline figure)
MMK lakh = total ÷ rate                   # rate = ¥ per 1 lakh
```

Receipt order: 値札価格 → 割引 → **割引後価格** (after discount price) → 消費税 → **小計** (shop total) → 代行手数料 → お支払い額

## Updating the MMK rate

Edit `config.json` — this is the only file you need to touch:

```json
{ "mmkRate": 3700 }
```

The app loads this on startup and sets the rate field automatically. On GitHub, click the
pencil icon on `config.json`, change the number, and commit — GitHub Pages will reflect the
update within about a minute.

You can also override the rate interactively in the gear panel; that overrides only your
current session (or a saved settings file) and does not change `config.json`.

## Remember your settings

In the gear panel, **Save** shares / downloads `jewellery-settings.txt` (your rate, tax, fee,
and the date the rate was last set). **Load** reads that file back, restoring the saved date
rather than overwriting it. The rate's set-date is stamped automatically when you change the
rate value, and displayed both in the settings panel and as a note under the total.

## Run locally

```
python3 -m http.server 8000
# open http://localhost:8000
```

The app must be served (not opened as `file://`) because `config.json` is fetched on startup.
If `config.json` is missing or unreachable the app falls back to the hardcoded default rate (3700).

## Install on iPhone

Open the published URL in Safari → **Share → Add to Home Screen**. It launches full-screen
with a gold-diamond icon and a dark status bar, like a native app.

## Notes

- Pure client-side: no build step, no backend, no tracking.
- No `localStorage` / `sessionStorage` — persistence is via the exported/imported settings file.
- The app icon, favicon, and PWA manifest are all embedded inline in `index.html` (no extra assets).
- See `CLAUDE.md` for full architecture, decisions, and the development backlog.
