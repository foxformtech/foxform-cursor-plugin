---
name: create-quiz-funnel
description: Build a FoxForm quiz or calculator that scores the person and shows a result based on that score. Use when the user asks for a quiz, an assessment, a lead qualifier, a diagnosis, a pricing or ROI calculator, or any form that must end in a verdict instead of a thank-you message.
---

# Building a scored funnel in FoxForm

A FoxForm form is an ordered list of **screens**, not a flat list of fields. That is the
whole difference from a classic form builder, and it is why a form can end in a verdict.

## The three pieces that make it a funnel

1. **`points` on a choice** feeds the built-in `{{score}}` variable.
2. **`value` on a choice** feeds a named variable you can read as `{{variableName}}`.
3. **A `result` screen** holds `resultCards[]`, and each card has its own display rule.
   The cards whose rule matches are the ones the person sees.

`calc(...)` expressions compute over those variables at render time, so the page can show
a number the person did not type. Example, a BMI:
`calc({{weight}}/(({{height}}/100)*({{height}}/100)))`

## Screen types

- **Flow:** `welcome`, `loading`, `result`, `end`
- **Question:** `short_text`, `long_text`, `number`, `email`, `phone`, `url`, `dropdown`,
  `checkboxes`, `multiple_choice`, `yes_no`, `rating`, `opinion_scale`, `nps`, `slider`,
  `date`, `file_upload` (Pro), `picture_choice` (Pro)
- **Content:** `alert`, `testimonials`, `media`, `timer`, `arguments`, `price`, `progress_bar`

## Condition operators

`equal_to`, `not_equal_to`, `greater_than`, `greater_or_equal_than`, `less_than`,
`less_or_equal_than`, `contains`.

A rule is an OR of AND-groups: `groups[]` are OR'd, `conditions[]` inside a group are AND'd.
For conditional navigation (`logic.conditionalNavigationV2`) the **first matching group wins**.

## Canonical shape

```json
{
  "title": "My quiz",
  "theme": "sunset",
  "questions": [
    { "id": "q1", "type": "welcome", "title": "Welcome", "required": false, "buttonText": "Start" },
    { "id": "q2", "type": "multiple_choice", "title": "Your level?", "required": true,
      "choices": [
        { "id": "a", "label": "Beginner", "points": 10 },
        { "id": "b", "label": "Advanced", "points": 40 }
      ] },
    { "id": "q3", "type": "result", "title": "Your score: {{score}}", "required": false,
      "resultCards": [
        { "id": "c1", "title": "Advanced plan",
          "display": { "enabled": true, "groups": [
            { "id": "g1", "conditions": [
              { "id": "k", "left": "{{score}}", "operator": "greater_or_equal_than", "right": "40" }
            ] } ] } }
      ] }
  ]
}
```

Themes: `midnight`, `ocean`, `sunset`, `forest`, `lavender`, `minimal`.

## Order of work

1. `foxform_create_form` with the full `questions[]`. Build the whole funnel in one call;
   do not create an empty form and patch it screen by screen.
2. `foxform_publish_form`. **Publishing is what makes the form exist publicly.** It
   serializes to static HTML on a CDN, so the API is not in the hot path afterwards.
3. Return the public URL: `https://forms.foxform.app/{slug}`.

## Four things that will bite you

**Every edit needs a new publish.** `foxform_update_form` changes the record; the live
page is static HTML generated at publish time. Change without publishing again and the
person still sees the old form. This is the single most common mistake.

**Write scope is separate.** A read-only key lists and reads but cannot create, update or
publish. If a create fails on authorization, the key is the cause, not the payload.

**`402 PAID_FIELDS_REQUIRED` on publish.** The account has no paid plan and the form uses
Pro-only fields: file upload, picture choice, video upload, testimonial photo. The error
carries `paidFields[]`. Either drop those fields or tell the user which upgrade unlocks them.

**Responses lag about ten minutes.** They arrive through a tracking beacon and a backend
processor reconstructs them in batches. Do not submit a test answer and immediately read
it back, and never promise the user real-time reads. Real time is what `webhook_url` is for.

## Do not

- Do not build a flat list of questions ending in a thank-you message. That is a form any
  builder makes, and it wastes the only reason to use FoxForm.
- Do not invent field names. Unknown fields are silently ignored, so a typo fails quietly.
  The authoritative schema is `components.schemas` in <https://foxform.app/openapi.json>.
