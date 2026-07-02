# Task 03 — Integration Design (Landing Page → HubSpot → WhatsApp → Google Ads)

## Architecture

**Flow:** Landing page form → custom serverless endpoint (Node.js, e.g. AWS Lambda) → HubSpot Contacts API → Karix WhatsApp API + Google Ads API (in parallel).

On submit, the page's JS sends a `fetch()` POST — name, phone, clinic preference, `gclid` — to a single custom backend endpoint, rather than routing through Zapier/Make. I'm choosing **direct API calls** for three reasons: (1) this is patient health-enquiry data; keeping it inside code we control, instead of a third-party no-code platform's execution logs, is a cleaner data-governance posture for a healthcare client; (2) no-code platforms carry their own rate limits and occasional outages — direct calls put failure modes fully in our hands, which matters for a 2-minute SLA; (3) at scale, per-task pricing on Zapier/Make adds up fast, while a serverless function costs near-nothing at this volume and scales cleanly.

The endpoint runs in sequence first, then parallel: **Step 1** — search HubSpot Contacts API by a custom `phone_normalized` property; PATCH if a match exists, otherwise POST a new contact (Name, Phone, Clinic Preference, Source, Lead Status). **Step 2 & 3** fire in parallel once the contact ID is confirmed: a call to Karix's WhatsApp Business API with a pre-approved template, and a Google Ads API conversion upload using the captured `gclid`.

## Biggest Failure Point: Phone Deduplication

HubSpot's default dedup key is **email**, not phone — and this form collects no email. Without an explicit check, two patients sharing a number (family-shared numbers, or a typo'd digit) will either create duplicate contacts or silently overwrite an existing patient's name.

**Fallback:** the endpoint always searches HubSpot on `phone_normalized` before writing. If a match exists, it **appends** a new enquiry note with a timestamp and clinic preference instead of overwriting the Name field, so a same-number-different-patient case is logged as a fresh enquiry rather than corrupting the record.

## Monitoring the 2-Minute WhatsApp SLA

What could break it: Karix downtime, an expired WhatsApp template, or cold-start latency on the serverless function under load. Each request logs a timestamp at lead capture and at confirmed Karix delivery; a scheduled check compares the two and fires a Slack alert plus a HubSpot follow-up task for anything that fails or exceeds the window, so no lead is silently missed.
