---
template: consulting-pitch
lens: client
tone: peer, specific, value-first
length: 130–200 words
---

# Consulting Pitch — example

For a contact at a company you'd like to consult for. Different from cold-pitch: more credibility-establishing, more concrete on what you'd actually do.

## Variables

- `{{contact_first_name}}`
- `{{company_name}}`
- `{{specific_observation}}` — concrete signal that you'd be useful (problem they've talked about, role they're hiring for, project they've shipped).
- `{{your_relevant_artifact}}` — short proof point: link to repo / blog / case study / shipped product.
- `{{narrow_offer}}` — one specific thing you could do, with a rough scope (not a full SOW).
- `{{your_signature}}`

## Body

Subject: {{narrow_offer_topic}} — happy to chat if useful

Hi {{contact_first_name}},

Saw {{specific_observation}} — congrats / interesting / curious. I do work in this exact space; recent example: {{your_relevant_artifact}}.

If {{narrow_offer}} would be useful, I have capacity for a short engagement (rough shape: {{scope_outline}}). Happy to share more detail or to leave it here if the timing's off.

If you'd rather chat than read a wall of text — 20 minutes anytime next week, your call.

{{your_signature}}

## Notes for the agent

- Lead with their problem, not your offering.
- Offer one narrow scope, not a menu.
- Include a real artifact link — never invented.
- If consulting brand differs from personal email, send from the consulting brand (see ground-truth `freelance-brand`).
