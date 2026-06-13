# 宝飾計算 · Jewellery Price Calculator

A single-file, installable web app that turns a Japanese price tag into the final
amount you pay — applying the 割 (wari) discount, consumption tax, and a buying-agent
handling fee — and shows the result in both **JPY** and **MMK (lakh)**.

## What it does
1. Enter the **price tag** (¥) and the **discount** in 割 (e.g. `6` = 6割 = 60% off).
2. Behind the **gear icon**: adjust consumption tax (default 10%), handling fee
   (default 5%), and the FX rate (default ¥3,800 per 1 lakh MMK).
3. Read the itemized receipt and the headline **You pay** total in ¥ and lakh MMK.

The math (all yen rounded per line so the receipt sums exactly):
```
subtotal = round(price × (1 − wari/10))   # discount applied
tax      = round(subtotal × tax%)          # on subtotal
fee      = round((subtotal + tax) × fee%)  # on tax-inclusive amount
total    = subtotal + tax + fee
MMK lakh = total ÷ rate                     # rate = ¥ per lakh
```

## Run it
Open `jewellery-calculator.html` in any browser. To serve locally:
```
python3 -m http.server 8000
# open http://localhost:8000/jewellery-calculator.html
```

## Remember your settings
In the gear panel, **Save** downloads `jewellery-settings.txt` (your rate, tax, fee,
and the date the rate was set). **Load** reads that file back. The rate's set-date is
stamped automatically when you change the rate, and preserved when you reload a file.

## Deploy to GitHub Pages
1. Create a repo and add this folder.
2. Settings → Pages → deploy from `main`.
3. Visit the published URL (rename the file to `index.html` if you want a clean root URL).

## Install on iPhone
Open the published URL in Safari → **Share → Add to Home Screen**. It launches
full-screen with a gold-diamond icon and a dark status bar, like a native app.
Offline, it still works (fonts fall back to system fonts).

## Notes
- Pure client-side: HTML + CSS + vanilla JS, no build step, no backend, no tracking.
- The app icon, favicon, and PWA manifest are embedded in the file (no extra assets).
- See `CLAUDE.md` for full architecture, decisions, and the development backlog.
