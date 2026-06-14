# TMC Legal — Letter Builder (Azure Static Web App)

Client-side document generator. A button on the SharePoint case record opens
this app with case fields in the URL query string; the app fills a Word master
template (`.docx`) entirely in the browser and downloads the finished document.
No Power Automate, no Python, no server-side code, no Graph secret.

Successor route to the P102 Python/PA generation pipeline (project 102, WS-E).

## How it works

1. SharePoint Infowise button opens `builder.html?MatterName=...&FeeEarner=...&...`
2. The page reads the query string and pre-fills the form.
3. User checks the fields and clicks **Generate & download**.
4. `docxtemplater` (browser build) fills the `[[Placeholder]]` tokens in the
   master and the browser downloads the completed `.docx`.

Split-run `[[ ]]` placeholders are handled natively by docxtemplater (proven on
the real 1MB Formal Service HJA master and the Memo master — see project
de-risk note, 2026-06-14).

## Current template

This build is wired for **InterPartesMemoOfAdvice** (6 fields, all short, no
addresses — ideal first target). Placeholder → URL param names:

| Placeholder | URL param | Source (SP / .md)        | Required |
|-------------|-----------|--------------------------|----------|
| MatterName  | MatterName| Client Surname           | Yes      |
| FeeEarner   | FeeEarner | Fee Earner               | Yes      |
| Draftsman   | Draftsman | Assigned To              | Yes      |
| CaseRef     | CaseRef   | Client Reference         | Yes      |
| TMCRef      | TMCRef    | Our Reference            | Yes      |
| Date        | Date      | derived (today) if absent| No       |

## Structure

```
index.html                     landing page (button-launched app, nothing to do here)
builder.html                   the builder UI + fill logic
masters/                       Word master templates (deployed as static assets)
  InterPartesMemoOfAdvice.docx
vendor/                        pinned browser builds (no CDN)
  pizzip.min.js
  pizzip-utils.min.js
  docxtemplater.js
staticwebapp.config.json       SWA routing (anonymous, SPA fallback)
.github/workflows/             Azure SWA deploy
```

## Data delivery

URL params (staged middle path). Fine for short, clean fields like the memo's.
For address-heavy templates (Formal Service: long opponent addresses, line
breaks, `&`) the planned upgrade is Graph-fetch-by-ID via an Azure Function —
not yet built. See project de-risk note.

## Deploy

1. Create a new Azure Static Web App (separate from ack-email-builder).
2. In GitHub repo settings, add its deploy token as secret
   `AZURE_STATIC_WEB_APPS_API_TOKEN_LETTER_BUILDER`.
3. Push to `main` — the workflow uploads the site.
4. Point the SharePoint Infowise button at `<swa-url>/builder.html` with the
   field tokens appended as query params (single `[Field]` token style, as
   proven on the Ack Email Builder).

## Adding more templates

This first cut is single-template. Next iteration: drive template choice + the
placeholder/param map from config (e.g. a `?template=` param + a small JS map),
reusing the registry.json field definitions from the Python project.
