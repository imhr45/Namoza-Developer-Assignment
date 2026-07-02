# Namoza Developer Assignment — OrthoNow

**Submitted by:** [Himanshu Ranjan]
**Role applied for:** Developer
---

## Repo Structure

- **Task 1 — GTM Event Schema:** [`/Task-1/GTM_Event_Schema.md`](./Task-1/GTM_Event_Schema.md)
  Full event tracking table, 3-step booking form dataLayer JSON, GA4 Funnel Exploration setup, and Google Ads conversion recommendation.

- **Task 2 — Landing Page:** [`/Task-2/index.html`](./Task-2/index.html)
  Single self-contained HTML file (HTML + CSS + vanilla JS, no frameworks).
  **Live version:** https://imhr45.github.io/Namoza-Developer-Assignment/Task-2/index.html
  **PageSpeed Insights (Mobile):** 100 Performance / 91 Accessibility / 100 Best Practices / 100 SEO — see [`/Task-2/pagespeed-screenshot.png`](./Task-2/pagespeed-screenshot.png.png)

- **Task 3 — Integration Design:** [`/Task-3/Integration_Design.md`](./Task-3/Integration_Design.md)
  Written architecture for connecting the landing page to HubSpot, WhatsApp (Karix), and Google Ads.

---

## Loom Walkthrough

**[https://www.loom.com/share/7cf6113686a8415f9559c10fa2d8bd2b]**

Covers: GTM schema decisions (2 min) → live demo of the landing page form firing the dataLayer push in browser console (3 min) → integration architecture answer (3 min).

---

## How to Run Task 2 Locally

No build step or server needed — it's a single static file.

1. Download `/Task-2/index.html`
2. Open it directly in any browser
3. Open DevTools Console (F12) and submit the form to see the `consultation_form_submitted` event pushed to `window.dataLayer`
