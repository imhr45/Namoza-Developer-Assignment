# Task 01 — GTM Event Schema (OrthoNow)

## 1. Complete Event Schema

| Event Name | Trigger Type (GTM) | Key Parameters | GA4 Report / Audience it feeds |
|---|---|---|---|
| `booking_step_complete` | Custom Event (dataLayer push — see §2) | `step_number`, `step_name`, `clinic_location`, `specialty` | Funnel Exploration (booking funnel), "Started Booking" audience |
| `booking_confirmed` | Custom Event (dataLayer push on step 3 success) | `clinic_location`, `specialty`, `preferred_date`, `booking_id` | Conversions report, "Converters" audience, **imported as Google Ads conversion** |
| `call_now_click` | GTM Click Trigger (Just Links / All Elements, matching `.call-now-btn` or `tel:` href) | `page_location`, `clinic_location` (if on a clinic page), `button_placement` (homepage/clinic-page/landing-page) | Engagement report, "High Intent — Call" audience |
| `whatsapp_click` | GTM Click Trigger (matches `wa.me` href) | `page_location`, `device_category`, `time_on_page` | Engagement report, "High Intent — WhatsApp" audience |
| `patient_guide_download` | GTM Form Submission Trigger (on the gated name+phone form) | `form_location`, `lead_name` (hashed if needed), `download_file` | Lead gen report, "Content Downloaders" audience, remarketing list |
| `clinic_page_view` | GTM Page View Trigger (condition: URL matches `/clinics/*`) | `clinic_name`, `clinic_city`, `page_referrer` | Pages & Screens report, per-city engagement |
| `blog_scroll_depth` | GTM Scroll Depth Trigger (25/50/75/90%) | `scroll_percentage`, `blog_title`, `blog_category` | Engagement report, "Engaged Readers" audience for content remarketing |

**Note on `call_now_click`:** Since this button repeats across homepage, 9 clinic pages, and the landing page, I'd use **one trigger** with a shared CSS class (`.call-now-btn`) rather than 9 separate triggers — and pass `clinic_location` dynamically via a Data Layer Variable or a DOM element attribute (e.g. `data-clinic="chennai-anna-nagar"`), so we don't maintain 9 near-duplicate tags.

---

## 2. Booking Form Funnel Tracking (3 Steps)

**Important context (this is the actual crux of this task):** GTM cannot natively detect "user completed step 1 of a JS-driven multi-step form." There's no DOM event for that. It can only listen to things that already happen in the browser — clicks, page loads, native form submits, scroll. A multi-step form is usually one single page where steps are shown/hidden with JS, so **there is no page load or native submit between steps for GTM to hook into**.

So the front-end developer has to explicitly push a dataLayer event **at the moment each step is completed**, and GTM's job is just to listen for that custom event via a **Custom Event trigger** and forward it to GA4.

### Who writes the code?
The **front-end developer** writes the `dataLayer.push()` call inside the form's JS (e.g. in the "Next" button's onClick handler, after validation passes). I (as the tag-management/martech person) define **exactly what the payload should look like** and brief the dev on **where** in the code it needs to fire. I don't write the form's business logic — but I do specify the contract.

### Step-by-step dataLayer pushes

**Step 1 — Location + Specialty selected** (fires when user clicks "Next" after step 1 fields are valid):
```json
{
  "event": "booking_step_complete",
  "step_number": 1,
  "step_name": "location_specialty_selected",
  "clinic_location": "Bengaluru - Indiranagar",
  "specialty": "Knee & Joint Care"
}
```

**Step 2 — Contact details entered** (fires when user clicks "Next" after step 2 fields are valid):
```json
{
  "event": "booking_step_complete",
  "step_number": 2,
  "step_name": "contact_details_entered",
  "clinic_location": "Bengaluru - Indiranagar",
  "specialty": "Knee & Joint Care",
  "preferred_date": "2026-07-15"
}
```

**Step 3 — Booking confirmed** (fires on successful backend confirmation, NOT just on button click — this matters because a failed API call shouldn't count as a conversion):
```json
{
  "event": "booking_confirmed",
  "step_number": 3,
  "step_name": "booking_confirmed",
  "clinic_location": "Bengaluru - Indiranagar",
  "specialty": "Knee & Joint Care",
  "booking_id": "ORTH-20260715-0042"
}
```

### GTM Trigger Setup
- One **Custom Event trigger** with Event name = `booking_step_complete` → fires a GA4 Event tag that reads `step_number` and `step_name` from dataLayer variables and sends them as event params.
- A **separate** Custom Event trigger for `booking_confirmed` (kept separate from step-tracking so it can be independently imported into Google Ads without accidentally counting step-1 as a conversion).

### Surfacing drop-off in GA4 Funnel Exploration
1. In GA4 → Explore → **Funnel Exploration**.
2. Build an **open funnel** (so we still see users who entered mid-way, e.g. from a retargeted ad):
   - Step 1: `booking_step_complete` where `step_number = 1`
   - Step 2: `booking_step_complete` where `step_number = 2`
   - Step 3: `booking_confirmed`
3. Turn on "Show elapsed time" to see how long users take between steps — useful for spotting if step 2 (contact details) is where people hesitate (common in healthcare — people don't want to give phone number yet).
4. Break down by `clinic_location` as a secondary dimension to see if drop-off is worse for specific clinics (could indicate availability/copy issues per location).

---

## 3. Conversion Action for Google Ads Import

**Recommended: `booking_confirmed`** (not `patient_guide_download`, not `call_now_click`, not `whatsapp_click`).

**Why this one over the others:**
- It's the only event that represents a **completed, backend-confirmed transaction** — not just intent. `call_now_click` and `whatsapp_click` only prove someone tapped a button; we don't know if the call/chat actually led anywhere, so importing those as the primary conversion would let Google's bidding algorithms optimise toward low-quality "taps" rather than real bookings.
- `patient_guide_download` is a good **micro-conversion** for remarketing audiences, but it's too far up-funnel to be the primary Smart Bidding signal — optimising toward it would fill the funnel with people who just want a free PDF, not people ready to book.
- `booking_confirmed` directly maps to what the business actually wants (a booked appointment), which is what you want Target CPA / Maximize Conversions bidding to chase.

*(Secondary recommendation: `call_now_click` can still be imported as a **secondary/observation-only** conversion action — not primary — since phone calls are a real channel in healthcare, but it shouldn't drive bidding on its own without call tracking/duration data attached.)*
