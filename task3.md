# Task 03 — Integration Design

## How I'd architect this end-to-end

On form submit, the front-end sends the payload directly to a serverless middleware layer — a small Node/Express function deployed on Vercel (or a Make.com scenario, if the team prefers low-code) — rather than calling HubSpot directly from the browser. This keeps the HubSpot private app token off the client entirely.

That middleware function does three things, in sequence:

1. **HubSpot Contacts API** — calls `PATCH /crm/v3/objects/contacts` to upsert the contact with Name, Phone, Clinic Preference, Source = "Google Ads - Consultation Landing Page", and Lead Status = "New Enquiry".
2. **Karix WhatsApp Business API** — sends the pre-approved confirmation template to the patient's number.
3. **Google Ads conversion** — fires the `consultation_form_submitted` conversion via the Google Ads API / Enhanced Conversions (or lets a GTM server-side container handle this leg if one is already in place).

**Why a direct API call via middleware, rather than Zapier or Make:** the 2-minute WhatsApp SLA needs low, predictable latency. Zapier's polling-based triggers on non-premium tiers can lag by several minutes, which risks blowing the SLA before the automation even starts. Make.com is closer to real-time but still adds an extra hop with less control over retry logic than a purpose-built function. A direct API call gives full control over timing, retries, and failure handling.

## Biggest failure point

**Phone-based deduplication.** HubSpot's native contact deduplication is keyed on **email**, not phone. Since this form only collects Name + Phone (deliberately minimal, per the requirements), two different patients submitting from the same number — or the same patient submitting twice with a slightly different name — will not be automatically merged. HubSpot will happily create duplicate contact records.

**Fallback:** before creating a contact, the middleware first queries HubSpot via `POST /crm/v3/objects/contacts/search`, filtering on the phone property, to check for an existing match. If found, it updates that record in place instead of creating a new one. This makes phone number the effective dedup key at the application layer, compensating for what HubSpot won't do natively.

If two patients genuinely share a phone number (e.g. a shared family line), the middleware appends the new name as a note/activity on the existing contact rather than overwriting the name field, so no enquiry gets silently lost.

## What could break the 2-minute WhatsApp SLA, and the fallback

Three realistic risks:

- **Karix API rate limits or downtime** — the vendor's WhatsApp API could throttle or go down during a traffic spike.
- **Serverless cold starts** — a function that hasn't run recently can take a few extra seconds to spin up, which matters when the budget is only 2 minutes.
- **Unapproved WhatsApp templates** — Meta requires template pre-approval; if the confirmation template isn't approved for this exact use case, the send is silently rejected rather than delayed.

**Fallback design:** the WhatsApp send is queued as a retryable background job — decoupled from the synchronous HubSpot write, not chained inside the same request — with up to 3 retries spread over 90 seconds. If all three attempts fail, the system alerts the ops team via a Slack webhook so a human can follow up manually. Decoupling this step means a Karix outage never blocks the HubSpot contact from being created, so the lead itself is never lost even if the WhatsApp confirmation is delayed.
