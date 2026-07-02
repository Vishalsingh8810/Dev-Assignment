# Task 01 — GTM Event Schema

## Event Schema Table

| Event Name | Trigger Type | Key Parameters | GA4 Report / Audience |
|---|---|---|---|
| `booking_step1_complete` | Custom Event (dataLayer push) | `step_number: 1`, `step_name: 'location_specialty_selected'`, `clinic_location`, `specialty` | Funnel Exploration – Step 1; Audience: "Booking Started" |
| `booking_step2_complete` | Custom Event (dataLayer push) | `step_number: 2`, `step_name: 'datetime_selected'`, `preferred_date`, `clinic_location` | Funnel Exploration – Step 2 |
| `booking_step3_complete` (= `appointment_confirmed`) | Custom Event (dataLayer push) | `step_number: 3`, `step_name: 'booking_confirmed'`, `clinic_location`, `specialty`, `patient_type` | Conversion event; primary Google Ads import |
| `call_now_click` | Click trigger (Just Links / All Elements, Click Classes = `.call-now`) | `page_location`, `clinic_location`, `click_source` (homepage / clinic page / landing page) | Engagement report; secondary conversion |
| `whatsapp_chat_open` | Click trigger on floating widget | `page_location`, `page_type` | Engagement audience |
| `patient_guide_download` | Form Submission trigger (gated form) | `form_name: 'patient_guide'`, `lead_source`, `page_location` | Lead magnet conversion |
| `clinic_page_view` | History Change / Page View, condition on URL path | `clinic_name`, `page_path` | Location-interest audience (9 clinics) |
| `blog_scroll_depth` | Scroll Depth trigger (25/50/75/90%) | `scroll_threshold`, `article_title` | Engaged-reader audience for remarketing |

---

## Booking Form (3-Step) — dataLayer Push per Step

GTM's Custom Event trigger only fires off a `dataLayer.push()` that already exists in the page — it has no native way to detect a multi-step form's internal state changes on its own. The front-end developer owns pushing these events at the exact moment each step completes (e.g. on the "Next" button click after a step validates). GTM's role is to listen for `event: 'booking_step_complete'` and route the parameters into GA4 as a custom event, with `step_number` / `step_name` registered as GA4 event parameters.

```json
// Step 1: location + specialty selected
{
  "event": "booking_step_complete",
  "step_number": 1,
  "step_name": "location_specialty_selected",
  "clinic_location": "{{clinic name}}",
  "specialty": "{{specialty selected}}"
}
```

```json
// Step 2: date/time preference entered
{
  "event": "booking_step_complete",
  "step_number": 2,
  "step_name": "datetime_selected",
  "clinic_location": "{{clinic name}}",
  "preferred_date": "{{selected date}}"
}
```

```json
// Step 3: booking confirmed
{
  "event": "booking_step_complete",
  "step_number": 3,
  "step_name": "booking_confirmed",
  "clinic_location": "{{clinic name}}",
  "specialty": "{{specialty selected}}"
}
```

### Surfacing step-level drop-off in GA4

Build a **Funnel Exploration** with three open, sequential steps, each defined by:
`event_name = booking_step_complete` AND `step_number = 1 / 2 / 3` (in order).

GA4 will automatically compute and display the percentage drop-off between each step, so the performance marketing team can see exactly where users abandon the booking flow (e.g. losing 40% between step 1 and step 2 signals a friction point in date/time selection).

---

## Conversion Action to Import into Google Ads

**Recommended: `booking_step3_complete` (appointment confirmed)** — not form-start or call-click events.

**Why this one over the others:** It's the only event tied to an actual completed booking. Importing an earlier-funnel event (like step 1, or a call click) as the primary conversion would train Google Ads' automated bidding to optimize toward people who show early interest but never convert — this inflates lead *volume* while degrading lead *quality*, since the algorithm learns to find more of the wrong kind of click. `booking_step3_complete` ties spend directly to the business outcome that matters: a confirmed appointment.

`call_now_click` can be added later as a **secondary/observed conversion** for visibility, but should not carry the "primary" flag Google Ads uses for bid optimization.
