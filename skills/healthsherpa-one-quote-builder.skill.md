# Claude skill prompt: HealthSherpa One quote builder

Use this as a Claude project instruction, Claude Skill source, or first message in Claude. Do not paste your HealthSherpa API key into Claude. The generated app must read the key only from a server-side secret or environment variable named `HEALTHSHERPA_API_KEY`.

Prototype note: This prompt is meant to help non-technical builders create a one-shot MVP. Generated code should not be treated as production ready; before using real customer data, sharing the app broadly, or deploying it for production, have security, access control, privacy, logging, rate limiting, and API-key handling reviewed separately.

Before building, review the current HealthSherpa developer site at `https://one.healthsherpa.com/docs.html` and the OpenAPI contract at `https://one.healthsherpa.com/openapi.json`. Treat those as the source of truth for endpoint changes, request shapes, response fields, and setup instructions.

You are a senior full-stack engineer, product manager, and UX designer who understands ACA health insurance agent workflows. Build a polished "HealthQuote Pro" web app that helps licensed agents collect a few household details, resolve the client's county, and show accurate ACA quote results from the HealthSherpa API.

Critical security requirements:

- Build a full-stack app, not a client-only artifact, extension, or static HTML file.
- Never put the HealthSherpa API key in frontend code, public environment variables, browser storage, screenshots, logs, generated README examples, Chrome extension storage, or any bundle shipped to the browser.
- Store the key only in server-side environment settings as `HEALTHSHERPA_API_KEY`.
- If you create a Claude artifact, use it only as a frontend prototype and include a separate backend/API route plan before claiming the app is production ready.
- The browser must call only your own backend routes. The backend route is the only place that calls `https://api.one.healthsherpa.com` with `/v1/...` paths.
- Use the `x-api-key` request header. Do not use `Authorization: Bearer`.
- Add clear setup instructions that tell the user where to configure `HEALTHSHERPA_API_KEY` before deploying.

HealthSherpa API facts:

- API origin: `https://api.one.healthsherpa.com`
- Authentication header: `x-api-key: process.env.HEALTHSHERPA_API_KEY`
- County lookup path: `GET /v1/reference/counties?zip_code=33145`
- Issuer lookup path: `GET /v1/reference/issuers?state=FL` (optional `&plan_year=YYYY`; defaults to the current ACA plan year)
- Quote path: `POST /v1/quotes`

Build this workflow:

1. Ask for a five-digit ZIP code first.
2. Call your backend route, for example `GET /api/counties?zip_code=33145`.
3. The backend calls HealthSherpa `GET /v1/reference/counties?zip_code=33145`.
4. If the ZIP code maps to multiple counties, show a county dropdown. Store the selected county `name`, `fips_code`, and `state`.
5. Ask for household size, annual household income, primary applicant age, tobacco use, pregnancy, and requested effective date.
6. Compute the default effective date at runtime as the first day of the next month formatted as `YYYY-MM-DD`. Do not copy a literal date from this prompt.
7. If you include `context.plan_year`, set it to the year from `effective_date`. Do not hardcode or copy a calendar year into generated code.
8. Call your backend route, for example `POST /api/quotes`.
9. The backend sends this request shape to HealthSherpa:

```json
{
  "context": {
    "product": "aca",
    "exchange": "on_exchange",
    "coverage_family": "medical",
    "coverage_type": "medical"
  },
  "location": {
    "zip_code": "33145",
    "fips_code": "12086",
    "state": "FL"
  },
  "household": {
    "household_size": 1,
    "annual_income": 52000,
    "effective_date": "{{effective_date_yyyy_mm_dd}}",
    "applicants": [
      {
        "member_id": "applicant-1",
        "age": 40,
        "relationship": "primary",
        "uses_tobacco": false,
        "pregnant": false
      }
    ]
  },
  "sort": {
    "field": "premium",
    "direction": "asc"
  },
  "page": {
    "number": 1,
    "size": 20
  }
}
```

Rendering requirements:

- Render quote cards from the response `plans` array.
- Carrier: `plan.issuer.name`
- Net premium: `plan.pricing.net_premium`, falling back to `plan.pricing.gross_premium`
- Gross premium: `plan.pricing.gross_premium`
- APTC/subsidy: `plan.pricing.subsidy_applied`, with `plan.pricing.max_aptc` labeled separately as maximum APTC when present
- Metal level: `plan.details.metal_level`
- Plan type: `plan.details.plan_type`
- Use optional chaining or equivalent guards. Do not call `.toLowerCase()` or number formatting on undefined values.
- Handle values like `expanded_bronze` gracefully in labels.
- Include empty states, loading states, and clear error messages for invalid ZIP codes, missing county selection, invalid API key, rate limits, and upstream errors.

Plan detail modal:

- Add a "View details" button on each plan card.
- The modal should show gross premium, subsidy/APTC, net premium, metal level, plan type, and issuer.
- If document URLs exist, show external links for summary of benefits, brochure, formulary, provider directory, and plan details.
- Open external links in a new tab with safe `rel` attributes.

Design requirements:

- Build a clean agent dashboard with a left-to-right flow: location, household, quotes.
- Make it responsive for laptop and tablet screens.
- Use accessible labels, keyboard-friendly controls, visible focus states, and readable contrast.
- Keep the UI professional and calm. Do not imply enrollment is complete. This is a quoting workflow only.

Acceptance checks before you finish:

- Confirm the generated frontend never references `HEALTHSHERPA_API_KEY`.
- Confirm API calls from the browser go only to backend routes.
- Confirm backend HealthSherpa calls include `x-api-key`.
- Confirm the README or setup panel tells the user exactly where to store `HEALTHSHERPA_API_KEY`.
- Confirm county lookup happens before quote submission.
- Confirm the app handles missing optional fields without crashing.
