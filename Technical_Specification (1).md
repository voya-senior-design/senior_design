# Voya - Technical Specification

Team: Tessneem Khalil and Salwa Syed | Advisor: Dr. Jillian Aurisano

**Traces to:** [Requirements_Document.md](Requirements_Document.md), **version 1.0 (2026-10-29)**. Terms are defined in that document's glossary.

## 1. Version and Change Log

| Version | Date | Change | Author |
|---|---|---|---|
| 1.0 | 2026-10-29 | First version. Built from Requirements_Document.md v1.0 and Design D0 to D2. Includes the data-model clarification below and clarifies API response fields, exchange-rate caching, and climate refresh recovery. | Tessneem Khalil, Salwa Syed |

**Design clarification logged here.** D0 says an itinerary stores recommendations with its review, but the D1 entity list shows only `review`. This spec adds a `recommendations` text-array column on `itinerary` so the stored data matches the D0 `TripDraft`. D0 and D1 files were not edited.

## 2. Architecture Summary

Voya is a modular monolith. A React web client (C1) runs in the browser and talks to one Node.js/Express server over HTTPS and JSON. The server contains the API layer (C2) and four domain modules: Itinerary Service (C3), Cost Aggregation Engine (C4), Season Search Service (C5), and Feedback Console (C6). They call each other in-process. All of them read and write one PostgreSQL database (C7), and only the server modules touch it. Three outside services sit behind the server: an OpenID Connect identity provider (X1), a climate data API (X2), and an exchange-rate API (X3). The full picture is the D0 block diagram in [Design_Diagrams/](Design_Diagrams/).

**Deployment.** The built client is static files on Vercel or Netlify. The server and the PostgreSQL database run on a free or low-cost Render (or Railway) tier. The weekly climate refresh runs inside the server process, so there is one deployable server unit and no separate worker. On startup the server checks the most recent successful refresh and runs a missed refresh when needed. If the server is asleep, refresh waits until the next startup or scheduled run. Stored valid climate rows remain available with an age warning. Configuration comes from environment variables read once at startup.

**Data flow.** *Contributing a trip:* the Poster fills in a form (C1), which sends `POST /v1/itineraries` to C2. C2 checks the session cookie (identity from X1), passes the draft to C3, which validates it, gets dated exchange rates from X3, and writes the itinerary, days, and cost items in one transaction to C7. The saved ID comes back the same way. *Planning a trip:* the Planner picks a destination and month, C1 calls `GET /v1/cost-breakdown`, C2 calls C4, and C4 reads cost rows from C7 and returns averages, n, and a reliability label. Season search is the same shape through C5, which reads climate normals that were stored earlier by the weekly refresh from X2. Cost flags go C1, C2, C6, C7. See D0 data-flow diagrams A and B, and the D2 figures [fig_D2-4](Design_Diagrams/figures/fig_D2-4.png) (save flow with error paths) and [fig_D2-1](Design_Diagrams/figures/fig_D2-1.png) (climate refresh and fallback).

## 3. Interfaces and Data Formats

### 3.1 Component boundaries (D0 interfaces I1 to I13)

Additional typed definitions are in the D0 interface table. This table gives each boundary's protocol, format, and error behavior. Errors reach the client as the common error body in section 3.3.

| Boundary | From to | Protocol and format | Inputs | Outputs | Error behavior |
|---|---|---|---|---|---|
| I1 | User to C1 | Browser events and form fields | Trip entry, selections, actions | Rendered view, field errors | Keeps the draft, shows validation or a retry message. |
| I2 | C1 to C2 | HTTPS, JSON, session cookie | Operation, payload | Status, data or error | 400, 401, 403, 422, 503. C1 shows the error and keeps entered data. |
| I3 | C2 to X1 | OpenID Connect over HTTPS | Authorization code, state, verifier | ID token, subject, expiry | Bad issuer, audience, expiry, or state gives 401; provider outage gives 503. No writes allowed. |
| I4 | C2 to C3 | In-process call, typed objects | Operation (create, read, remix, update, delete), user ID, draft or itinerary ID | Itinerary or saved ID, field errors | 422, 403, 404 mapped by C2. No partial save. |
| I5 | C2 to C4 | In-process call | Destination ID, month | Average total, categories, n, reliability, nearest month | Month out of range gives 422. n of 0 returns `no_data`. Storage failure gives 503. |
| I6 | C2 to C5 | In-process call | Month | Destinations with rating, temperature, rain days, data-as-of | 422 for a bad month. Uses last valid rows with an age label. 503 if none. |
| I7 | C2 to C6 | In-process call | User ID, operation (submit, list, review), breakdown key, n, reason, flag ID | Flag ID, status, flags | 422 bad reason, 403 not a Reviewer, 404 missing flag, 409 stale status. |
| I8 | C3 to C7 | Parameterized SQL, transaction | Itinerary, days, cost items | Rows or saved ID, committed flag | Roll back on failure; C3 returns 503. Remix keeps the source unchanged. |
| I9 | C4 to C7 | Parameterized SQL, read | Destination ID, month, 24-month cutoff | Cost rows | n of 0 on empty rows. Failure gives 503. |
| I10 | C5 to C7 | Parameterized SQL, transaction | Read by month, or upsert of climate rows | Normals, saved count | Keeps prior valid rows on a failed refresh. Partial upsert rolls back. |
| I11 | C6 to C7 | Parameterized SQL, transaction | Insert, list, or update a flag | Flag ID, flags, saved flag | Roll back and 503. Never claims a flag was saved when it was not. |
| I12 | C5 to X2 | HTTPS, provider JSON mapped to Voya fields, weekly | Destination, month or all months | Normals, source, as-of | Validates units and ranges. Timeout, bad data, or rate limit keeps old rows and retries next run. No per-request call. |
| I13 | C3 to X3 | HTTPS, provider JSON mapped to Voya fields | Base currency, USD, rate date | Rate, effective date, source | Uses the cache first. For an uncached rate, 3 second timeout per attempt and up to 2 retries, then 503 with nothing written. |

### 3.2 API endpoint table

All paths are under `/v1`. All bodies are JSON. "Signed in" means a valid session cookie.

| Endpoint | Via | Auth | Request | Success | Errors | Req |
|---|---|---|---|---|---|---|
| `GET /auth/login` | I3 | none | none | 302 to X1 with state and PKCE challenge | 503 | REQ-13 |
| `GET /auth/callback` | I3 | none | `code`, `state` | 302 and session cookie (8 h) | 400, 503 | REQ-13 |
| `POST /auth/logout` | C2 | signed in | none | 204, cookie cleared | 401 | REQ-13 |
| `GET /cost-breakdown` | I5 | optional | `destination_id`, `month` (1-12) | 200 breakdown (3.3) | 422, 503 | REQ-01, 02, 03 |
| `GET /season-search` | I6 | optional | `month` (1-12) | 200 `{destinations:[{id, season_rating, temp_c, rain_days, data_as_of, stale_warning}]}` | 422, 503 | REQ-09, 10 |
| `POST /itineraries` | I4, I8, I13 | signed in, `Idempotency-Key` header | TripDraft (3.3) | 201 `{saved_id, status:"saved"}` | 400, 401, 422, 503 | REQ-04, 05 |
| `GET /itineraries/{id}` | I4 | optional | none | 200 itinerary with days and costs | 404 | REQ-15 |
| `PATCH /itineraries/{id}` | I4 | owner | `{visibility?, review?, days?}` (`days` only while private) | 200 `{id, visibility}` | 401, 403, 404, 409, 422, 503 | REQ-06, 07 |
| `DELETE /itineraries/{id}` | I4 | owner | none | 204 | 401, 403, 404 | REQ-06 |
| `POST /itineraries/{id}/remix` | I4 | signed in | none | 201 `{saved_id}` | 401, 404, 503 | REQ-08 |
| `POST /cost-flags` | I7, I11 | signed in | `{breakdown_key:{destination_id, month}, n, reason}` | 201 `{flag_id, status:"new"}` | 401, 422, 429, 503 | REQ-11 |
| `GET /admin/cost-flags` | I7 | Reviewer | `status?`, `limit?` | 200 `{flags:[...]}` | 401, 403, 422, 503 | REQ-12 |
| `PATCH /admin/cost-flags/{id}` | I7 | Reviewer | `{status, expected_status, resolution_note}` | 200 `{flag_id, status}` | 401, 403, 404, 409, 422, 503 | REQ-12 |
| `GET /healthz` (no prefix) | C2 | none | none | 200 `{status:"ok", db:"up"}` | 503 `{db:"down"}` | NFR-09 |

Versioning: adding an optional field is not breaking. Removing a field, changing a type, or changing an enum value needs a new version prefix.

### 3.3 Data formats

**TripDraft (request body of `POST /itineraries`), JSON Schema:**

```json
{
  "type": "object",
  "required": ["destination_id", "start_date", "end_date", "visibility", "days"],
  "properties": {
    "destination_id": {"type": "string"},
    "start_date": {"type": "string", "format": "date"},
    "end_date": {"type": "string", "format": "date"},
    "visibility": {"enum": ["public", "private"]},
    "review": {"type": "string", "maxLength": 5000},
    "recommendations": {"type": "array", "items": {"type": "string"}},
    "days": {"type": "array", "minItems": 1, "items": {
      "type": "object", "required": ["date", "costs"],
      "properties": {
        "date": {"type": "string", "format": "date"},
        "costs": {"type": "array", "items": {
          "type": "object", "required": ["category", "amount", "currency"],
          "properties": {
            "category": {"enum": ["flights", "lodging", "food", "activities", "transport", "other"]},
            "amount": {"type": "string", "pattern": "^[0-9]+(\\.[0-9]{1,2})?$"},
            "currency": {"type": "string", "pattern": "^[A-Z]{3}$"}}}}}}}}
}
```

Each cost item's exchange-rate date is the `date` of its containing day. The server also checks what this schema does not enforce: `end_date` is on or after `start_date`, the trip is 365 days or less, each amount is from 0 to 1,000,000, and the destination and currency are known.

**Cost breakdown (200 response of `GET /cost-breakdown`):** `{"average_total_usd": 842.50, "categories": [{"category": "flights", "average_usd": 310.00, "min_usd": 210.00, "max_usd": 480.00}], "n": 5, "reliability": "reliable", "nearest_month": null}`. `reliability` is `reliable`, `limited_data`, or `no_data`. A null `average_total_usd` means no observations, not a zero-cost trip. When `n` is 0, the client displays the no-data message required by REQ-03, with `nearest_month` if one exists, and does not display an empty breakdown. The API returns `nearest_month` only for `no_data`.

**Error body (all errors):** `{"code": "VALIDATION_ERROR", "message": "Check the trip dates.", "fields": {"end_date": "Must be on or after start_date."}}`. Codes used: `VALIDATION_ERROR`, `UNAUTHENTICATED`, `FORBIDDEN`, `NOT_FOUND`, `INVALID_TRANSITION`, `IMMUTABLE_WHEN_PUBLIC`, `RATE_LIMITED`, `CLIMATE_DATA_UNAVAILABLE`, `SERVICE_UNAVAILABLE`. No token, SQL error, or stack trace is ever included.

### 3.4 Database schema (C7, PostgreSQL)

| Table | Columns | Keys and indexes |
|---|---|---|
| `user` | `id` uuid, `oidc_subject` text, `display_name` text, `email` text, `role` enum(user, admin), `created_at` timestamptz, `deleted_at` timestamptz null | PK `id`; unique `oidc_subject`. Role `admin` is the Reviewer role and is set only by a direct database update. |
| `destination` | `id` text (slug), `name` text, `country_code` char(2), `created_at` | PK `id` |
| `itinerary` | `id` uuid, `user_id`, `destination_id`, `start_date` date, `end_date` date, `visibility` enum(public, private), `review` text (max 5,000), `recommendations` text[], `created_at` | PK `id`; FK `user_id`, `destination_id`; index `(destination_id, start_date)` |
| `day` | `id` uuid, `itinerary_id`, `date` date | PK `id`; FK `itinerary_id` (cascade delete) |
| `cost_item` | `id` uuid, `day_id`, `category` enum, `amount_usd` numeric(12,2) not null, `original_amount` numeric(12,2), `original_currency` char(3), `rate_date` date | PK `id`; FK `day_id` (cascade delete); index `(category)` |
| `climate_normal` | `destination_id`, `month` smallint 1-12, `temp_c` numeric(4,1), `rain_days` numeric(3,1), `source` text, `fetched_at` timestamptz | PK `(destination_id, month)` |
| `cost_flag` | `id` uuid, `user_id`, `destination_id`, `month` smallint, `observed_n` int, `reason` text (1-2,000), `status` enum(new, reviewed, dismissed), `resolution_note` text null, `created_at`, `updated_at` | PK `id`; FK `user_id`, `destination_id`; index `(status, created_at)` |
| `idempotency_key` | `user_id`, `key` text, `saved_id` uuid, `created_at` | PK `(user_id, key)`; rows older than 24 h are ignored |

Normal saves require a valid USD conversion and are all-or-nothing. The current release has no import endpoint, so `amount_usd` must not be null for newly saved cost items. The D2 notification outbox table is not built (Won't list, Requirements section 7).

## 4. Technology Stack

| Layer | Choice | Justification |
|---|---|---|
| Database | PostgreSQL | Itinerary, day, and cost rows must commit together (REQ-04, NFR-08). Indexed grouping meets NFR-01. Free tiers exist (CON-01). |
| Data access | Prisma ORM | Parameterized queries guard against injection (NFR-04). Migrations are generated for us (CON-02). |
| Server | Node.js with Express, one modular server | One language across the stack for two people (CON-02). Matches the single-server pattern in CON-07. Non-blocking I/O suits waiting on X1, X2, X3 (CON-08). |
| Client | React, static build | Runs in any browser (CON-03). Supports labeled, keyboard-operable controls (NFR-07, CON-06). |
| Dates | date-fns | Month and 24-month window math is error-prone (REQ-01, REQ-09). Saves writing it ourselves (CON-02). |
| Identity | OpenID Connect provider (X1) | Sign-in with no stored passwords (REQ-13, NFR-04, CON-04). |
| Sessions | Signed, HTTP-only, SameSite=Lax cookie, 8 h | No session table to build (CON-02). Meets NFR-04 and REQ-13. |
| Currency | Exchange-rate API (X3) with 24 h cache per currency and date | Rates change daily and are not worth maintaining (REQ-05). Cache protects NFR-03 and CON-08. |
| Climate data | Climate API (X2), weekly refresh into `climate_normal` | Keeps X2 off the interactive path (REQ-10, NFR-02, NFR-09, CON-08). |
| Hosting | Render or Railway (server and database), Vercel or Netlify (client) | Free or low-cost tiers (CON-01). Little DevOps work (CON-02). |
| Logging | One JSON line per event on stdout, with request ID and secret redaction | Needs no paid service (CON-01). Satisfies NFR-10 and NFR-04. |
| Configuration | Environment variables, validated at startup, frozen | No secrets in code or logs (NFR-04, CON-04). |

## 5. Known Constraints

| ID | Constraint | What it rules out | What it forces |
|---|---|---|---|
| CON-01 | Low-cost or free-tier services | Paid hosting, paid logging, paid queues | Free-tier Render, Vercel, and PostgreSQL; stdout logs; an in-process weekly job with startup catch-up instead of a worker service. |
| CON-02 | Two people, 14 weeks (2027-01-11 to 2027-04-23) | Anything beyond the Must and Should sets; hand-built auth or date code | Reused libraries and services (X1, X3, Prisma, date-fns); tasks T1 to T16 sized to fit; a 15 to 20 percent buffer per task. |
| CON-03 | Standard browser only | Native apps, special hardware | A responsive React client over HTTPS. |
| CON-04 | Store subject IDs, never passwords; credentials server-side | Password accounts, secrets in the client | OIDC sign-in, `oidc_subject` on `user`, secrets only in server environment variables. |
| CON-05 | No location traces, booking, or payment data; public itineraries omit private account details | Map tracking, bookings, payments; showing email or login ID on public pages | No such columns in the schema; public itinerary responses built from itinerary, day, and cost fields only. |
| CON-06 | Accessibility | Color-only reliability signals; mouse-only controls | Text reliability labels, labeled fields, keyboard-operable controls in C1. |
| CON-07 | One modular server and one relational store | Separately deployed microservices; a browser-only design | In-process calls between C2 and C3 to C6; one database for all data (D02). |
| CON-08 | X1, X2, X3 are outside our control | Calling X2 or X3 freely on the interactive path; trusting provider data unchecked | Stored climate normals with weekly refresh and startup catch-up, cached rates, validation of provider responses, and a 503 rather than a guessed value. |

## 6. Traceability Matrix

One row per REQ, NFR, and CON ID. Design components are named from D0 (boundaries and components), D1 (data model, algorithm, API contract), and D2 (behavior, state machines, edge cases). Test IDs are planned and provisional until the Week 10 test plan fixes them. T7 (concurrency tests) and T16 (performance validation) from the Task List are the test tasks behind TC-004, TC-008, TC-012, TC-016, TC-017, and TC-018.

| Req ID | Description | Design Component (D0/D1/D2) | Status | Test Case |
|---|---|---|---|---|
| REQ-01 | Cost breakdown with average and category ranges | C4 (D0 I5, I9); D1 Cost Breakdown Aggregation; `GET /v1/cost-breakdown` | Designed | TC-001 |
| REQ-02 | n and reliability label on every breakdown | C4 reliability tiers (D1, D0 D06); C1 results view (D2 2.9) | Designed | TC-002 |
| REQ-03 | No-data message and alternate month | C4 `findNearestMonth`, `nearest_month` (D1, D2 E-C4-01, E-C4-02) | Designed | TC-003 |
| REQ-04 | Save itinerary all-or-nothing | C3 `POST /v1/itineraries` (D1 Endpoint 2, D0 I4, I8); D2 Fig. D2-4; `idempotency_key` | Designed | TC-004 (T7) |
| REQ-05 | Convert costs to USD with stored rate | C3 with X3 (D0 I13); `cost_item` columns (D1); D2 Fig. D2-4, E-C3-06 | Designed | TC-005 |
| REQ-06 | Public/private switch and delete | C3 `PATCH`, `DELETE /v1/itineraries/{id}`; `itinerary.visibility`; D2 Fig. D2-3 | Designed | TC-006 |
| REQ-07 | Edit while private, publish with revalidation | C3 edit and publish steps (D2 2.4); D2 Fig. D2-3, E-C3-08 | Designed | TC-007 |
| REQ-08 | Remix into a private copy | C3 remix steps (D2 2.4); D0 D07; E-C3-04 | Designed | TC-008 (T7) |
| REQ-09 | Season search by month | C5 (D0 I6, I10); `GET /v1/season-search`; `climate_normal` (D2 2.2) | Designed | TC-009 |
| REQ-10 | Weekly climate refresh with fallback | C5 refresh job (D0 I12, D04); D2 Fig. D2-1, E-C5-02 to E-C5-04 | Designed | TC-010 |
| REQ-11 | Submit a cost flag | C6 `POST /v1/cost-flags` (D0 I7, I11); `cost_flag`; D2 2.3 | Designed | TC-011 |
| REQ-12 | Reviewer triage of flags | C6 admin endpoints; D2 Fig. D2-2, E-C6-03 | Designed | TC-012 (T7) |
| REQ-13 | Sign-in through OIDC, 8 h session | Auth/Session (D0 I3, D05; D2 2.5); `user` | Designed | TC-013 |
| REQ-14 | Keep draft after a failed save | C1 draft handling and `showRequestError` (D2 2.9, E-C1-02 to E-C1-04) | Designed | TC-014 |
| REQ-15 | Display itineraries by visibility | C3 `GET /v1/itineraries/{id}` (D0 I4; D2 2.4) | Designed | TC-015 |
| NFR-01 | Breakdown within 3 s at p95 | C4 and C7 indexes `(destination_id, start_date)`, `(category)` (D1 complexity analysis; D0 I9) | Designed | TC-016 (T16) |
| NFR-02 | Season search within 3 s at p95 | C5 reading stored `climate_normal` (D2 2.2; D0 I10) | Designed | TC-017 (T16) |
| NFR-03 | Save within 2 s at p95 for cached rates | C3 with cached X3 rates (D2 E-C3-06; D0 I13) | Designed | TC-018 (T16) |
| NFR-04 | Write auth, signed cookie, no secrets in logs | `authRequired`, session cookie (D2 2.5); Configuration (D2 2.7); log redaction (D2 2.8) | Designed | TC-019 |
| NFR-05 | Private itineraries stay private | C3 read rule and remix step 1 (D2 2.4); `404` not `403` | Designed | TC-020 |
| NFR-06 | First-time user tasks within time and rate | C1 screens for trip entry, search, and breakdown (Task T1; D2 2.9) | Planned | TC-021 |
| NFR-07 | Keyboard operable, labeled, text reliability | C1 (D2 2.9; D0 section 6) | Designed | TC-022 |
| NFR-08 | No partial itineraries after failures | C3 transaction (D0 D02; D2 Fig. D2-4, E-C3-07); C7 | Designed | TC-023 |
| NFR-09 | Degraded service when X2 or X3 is down | C5 fallback (D0 D04); D2 Fig. D2-1, E-C5-04, E-AU-04; `/healthz` | Designed | TC-024 |
| NFR-10 | Request ID on responses, no secrets in logs | Logging (D2 2.8, E-LG-01 to E-LG-04) | Designed | TC-025 |
| CON-01 | Low-cost services | D1 technology choices (hosting); D0 section 6; D2 2.6 optional, 2.8 | Designed | TC-026 |
| CON-02 | Two people, 14 weeks | Task List T1 to T16 and Timeline (Assignments #5 and #6); D0 section 6 module split | Designed | TC-027 |
| CON-03 | Standard browser | C1 Web Client (D0 block diagram, I1, I2) | Designed | TC-028 |
| CON-04 | No passwords, credentials server-side | Auth/Session (D2 2.5); `user.oidc_subject` (D2 2.1); Configuration (D2 2.7) | Designed | TC-029 |
| CON-05 | No location or payment data; omit private account details | D1 data model (no such fields); C3 public response shape (D2 2.4) | Designed | TC-030 |
| CON-06 | Accessibility | C1 (D2 2.9; D0 section 6) | Designed | TC-031 |
| CON-07 | One modular server, one relational store | D0 section 6 architecture pattern; C2 to C6 modules; C7 (D0 D01, D02) | Designed | TC-032 |
| CON-08 | External services are unreliable | C5 and C3 with I12, I13 (D0 D04, D07); D2 E-C5-04, E-C3-06 | Designed | TC-033 |

No row is orphaned. Design elements with no requirement: none, except the optional notification outbox in D2 section 2.6, which is on the Won't list and is not built.

## 7. Advisor Review Record

**Advisor:** Dr. Jillian Aurisano.

| Date | What was discussed | Changes it produced | Logged in |
|---|---|---|---|
| Pending actual advisor review | Not yet confirmed | No advisor-directed changes can be claimed before the meeting | To be completed after review |

The team should send both drafts at least three days before the deadline and bring the traceability matrix to the meeting. After the meeting, replace this pending entry with the real date, discussion, and resulting edits, then update the change logs. This record is intentionally not presented as a completed meeting.
