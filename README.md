# OrthoNow Digital Growth — Developer Assignment

Submission for the Developer - Position 1 assignment (Namoza / OrthoNow account).

--check for live Demo-----  https://orthonowdev.netlify.app/

## Repo Structure

```
├── README.md                    → this file
├── task1.md                     → GTM event schema (table + dataLayer JSON)
├── orthonow-final.html          → Task 2: working landing page (single file)
├── pagespeed-screenshot.png     → Task 2: PageSpeed Insights mobile score
└── task3.md                     → Task 3: integration architecture write-up
```

## Task 01 — GTM Event Schema
📄 [task1.md](./task1.md)

Full event schema table covering the booking form, call clicks, WhatsApp widget, patient guide download, clinic page views, and blog scroll depth — including the exact `dataLayer.push()` JSON for each step of the 3-step booking form, how step-level drop-off surfaces in a GA4 Funnel Exploration, and which single event should be imported into Google Ads as the primary conversion (and why).

## Task 02 — Landing Page Build
📄 [orthonow-final.html](./orthonow-final.html)

A single self-contained HTML/CSS/vanilla JS file — no frameworks, no external font/CDN requests, no server required.

**To view it:** download the file and open it directly in a browser.

**To verify the tracking:**
1. Open browser DevTools → Console.
2. Fill in a name and a valid 10-digit mobile number (starting with 6–9).
3. Submit the form — it swaps to the thank-you state without a page reload.
4. Run `window.dataLayer` in the console — you'll see the `consultation_form_submitted` event with `form_location` and `lead_source`. No personal data (name/phone) is sent into the dataLayer, since GA4's terms prohibit sending PII through Analytics.

**PageSpeed Mobile score:** see `pagespeed-screenshot.png` — [add your score once you've hosted the file and run it through PageSpeed Insights].

## Task 03 — Integration Design
📄 [task3.md](./task3.md)

Written answer (no code, per the brief) covering: the end-to-end architecture connecting the form to HubSpot, Karix WhatsApp, and Google Ads; the biggest failure point in the setup (HubSpot's email-based deduplication, and the phone-based fallback used instead); and what could break the 2-minute WhatsApp SLA plus how it's mitigated.

## Loom Walkthrough
🎥 [add your Loom link here once recorded]

Covers: GTM schema decisions (2 min) → live demo of the form firing the dataLayer push in console (3 min) → integration architecture answer (3 min).
