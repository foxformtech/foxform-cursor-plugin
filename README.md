# FoxForm plugin for Cursor

Connects [FoxForm](https://foxform.app) to Cursor through its hosted MCP server, so the
agent can build a form, publish it and read the answers back without anyone opening a
dashboard.

## What FoxForm is

A form, quiz and survey builder for forms that qualify instead of only collecting. Answer
options carry points into a score, the score decides which screen comes next and which
result the person sees at the end, and `calc()` expressions run over answers and variables
inside the page, so a form can finish with a quote, a diagnosis or a rating band instead of
a confirmation message.

Published forms are rendered as static HTML and served from a CDN.

## Install

```
/add-plugin foxform
```

## Setup

The plugin points at `https://mcp.foxform.app/mcp` over Streamable HTTP.

Listing the available tools works without authentication. Running any of them requires a
FoxForm API key, generated in the dashboard under **MCP & API**, which you paste into the
`FOXFORM_API_KEY` variable when installing. It is sent as the `x-api-key` header and never
leaves your machine except to `mcp.foxform.app`.

An account is free to create with no card, and the first 25 responses to a form are free
with no time limit.

## Tools

| Tool | What it does |
|---|---|
| `foxform_list_forms` | List the forms in the account |
| `foxform_get_form` | Get one form with its screens, logic and scoring |
| `foxform_create_form` | Create a form, quiz or calculator from a definition or a prompt |
| `foxform_update_form` | Update screens, questions, points, `calc()` and conditional navigation |
| `foxform_publish_form` | Publish as static HTML on the CDN and return the public URL |
| `foxform_unpublish_form` | Take a published form offline |
| `foxform_list_responses` | List responses for a form |
| `foxform_get_response` | Get one response with its answers, score and variables |
| `foxform_export_responses` | Export the responses of a form |
| `foxform_get_form_analytics` | Views, starts, completions and drop-off per screen |

## Things to know

Responses are collected through a tracking beacon and reconstructed on the backend roughly
every ten minutes, so a response written this second is not readable the next one. There is
no client-side write path.

Publishing a form that uses Pro-only fields (file upload, picture choice, video upload,
testimonial photo) returns `402 PAID_FIELDS_REQUIRED` when the account has no paid plan.

## Links

- Product: <https://foxform.app>
- API reference for machines: <https://foxform.app/llms-full.txt>
- OpenAPI: <https://foxform.app/openapi.json>

## License

MIT. See [LICENSE](./LICENSE).
