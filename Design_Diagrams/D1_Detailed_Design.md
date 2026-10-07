# Voya — D1 Detailed Design
Design D1 | Assignment #4 | Week 6
Team: Tessneem Khalil and Salwa Syed

## Header, Scope, and Conventions

**Goal:** Voya is a social platform for travelers to share trip reviews, recommendations, itineraries, and costs in one place. People can see where others traveled during different months and use their experiences to plan a trip that fits their budget.

This document details **C3 (Itinerary Service)**, specifically its data model, and **C4 (Cost Aggregation Engine)**, specifically its core algorithm. C5 (Season Search Service) and C6 (Feedback Console) are deferred to D2.

**Conventions:** In the entity-relationship diagram, rectangles are entities, lines are relationships, and cardinality is marked on both ends of each line (1 or *). Primary keys are marked (PK) and foreign keys are marked (FK). This is consistent with D0's convention that every diagram carries its own title, goal, and legend.

## Data Model

**Core component detailed:** C3 Itinerary Service / C7 Data Store (itinerary data)

**Entities:**
- **Itinerary**: id (PK), user_id (FK), destination_id, start_date, end_date, visibility (enum: public/private), review, created_at
- **Day**: id (PK), itinerary_id (FK), date
- **CostItem**: id (PK), day_id (FK), category (enum: flights/lodging/food/activities/transport/other), amount_usd (decimal), original_amount (decimal), original_currency (string), rate_date (date)

**Relationships:**
- Itinerary (1) — (*) Day: one itinerary has many days; each day belongs to exactly one itinerary.
- Day (1) — (*) CostItem: one day has many cost items; each cost item belongs to exactly one day.

**Structural decisions:**

| Decision | Reasoning |
|---|---|
| Itinerary is an entity, not a User attribute | It has its own lifecycle (created, remixed, queried by destination/month independent of the user) |
| Day is an entity, not a JSON field on Itinerary | UC-01's aggregation algorithm needs to filter/group cost rows by date range and category at the row level; embedding would block indexed queries |
| CostItem is an entity, not a Day attribute | Same reason: C4 groups cost rows by category across many itineraries, which requires row-level querying |
| Itinerary→Day is one-to-many, not many-to-many | A day on a trip only ever belongs to that one trip |
| Indexed fields: Itinerary(destination_id, start_date), CostItem(category) | These are exactly the fields I5/I9 filter and group by for the cost-breakdown query (UC-01) |

**Model choice:** Relational (not a document store). Per decision D02 in D0, itineraries/days/costs are linked records that need transactional writes so a partially-saved itinerary is never published, and UC-01's aggregation needs indexed filtering/grouping across many rows, which a relational model supports directly.

## Core Algorithms

**Algorithm: Cost Breakdown Aggregation** (serves interface I5, implements UC-01)

- **What it is / problem solved:** Given a destination and month, compute an average total cost and a per-category cost range from other travelers' completed itineraries for that destination in that month within the last 24 months, so a planner gets a transparent, sourced cost estimate.
- **Inputs:** destination_id (string), month (integer, 1–12)
- **Outputs:** average_total_usd (decimal, nullable), categories (array of {category: enum, average_usd: decimal, min_usd: decimal, max_usd: decimal}), n (integer), reliability (enum: reliable/limited_data/no_data), nearest_month (integer, nullable — only set when reliability = no_data)
- **Expected complexity:** At an estimated 500–2,000 itineraries in year one, an indexed query on (destination_id, month) returns a small bounded row set (well under 100 rows); grouping and summing by category is O(k) on that set, effectively instant. At 100x scale (~50,000–200,000 itineraries), the index keeps the filtered set small per destination/month, so the aggregation step stays O(k) on a small k rather than scanning the whole table — the index is what matters, not the total table size.
- **Why this approach:** Precomputing and caching every destination-month pair ahead of time was considered, but rejected for D1 — it adds a background recomputation/staleness-management job that a two-person team doesn't need yet at this data size. Computing on read is simpler to build and fast enough.
- **Edge cases:** zero matching itineraries (n=0 → reliability=no_data, suggest nearest month with n≥1, per D06); 1–2 matches (reliability=limited_data, number still shown); duplicate cost items in the same category on the same day (summed, not overwritten); a cost item missing its USD conversion due to a failed currency lookup (excluded from the aggregate, not treated as $0, per D03); ties in min/max across categories (handled naturally, no special case needed).

## Build-versus-Reuse Decisions

| Piece | Build or Reuse | Library/Service | License | Reason |
|---|---|---|---|---|
| Authentication/identity | Reuse | OpenID Connect provider (X1) | Provider terms | Avoids hand-rolling password storage/hashing; matches D05 |
| Currency conversion rates | Reuse | Exchange-Rate API (X3) | Provider terms | Rates change daily; not worth maintaining ourselves |
| Date/time handling | Reuse | date-fns | MIT | Date arithmetic is a common source of bugs; not worth hand-rolling |
| Database access layer | Reuse | Prisma ORM | Apache 2.0 | Provides migrations and parameterized queries, reducing boilerplate and injection risk |
| Cost aggregation algorithm | Build | — | — | Specific to Voya's reliability rules (n≥3/1–2/0); no existing library fits |
| Itinerary remix logic | Build | — | — | Specific business rule (copy without mutating the source); not something a library provides |

## API Contract

**Endpoint 1: GET /v1/cost-breakdown** (C4, implements I5/UC-01)
- Inputs: `destination_id` (string, required, must match a known destination), `month` (integer, required, 1–12)
- Output (200): `{ average_total_usd: decimal | null, categories: [{category: string, average_usd: decimal, min_usd: decimal, max_usd: decimal}], n: integer, reliability: "reliable"|"limited_data"|"no_data", nearest_month: integer | null }` — all monetary fields in USD
- Errors: `422 VALIDATION_ERROR` if month is out of range or destination_id is unknown; `503 SERVICE_UNAVAILABLE` if the data store can't be reached
- Example request: `GET /v1/cost-breakdown?destination_id=lisbon&month=6`
- Example response: `{"average_total_usd": 842.50, "categories": [{"category":"flights","average_usd":310.00,"min_usd":210.00,"max_usd":480.00}], "n": 5, "reliability": "reliable", "nearest_month": null}`

**Endpoint 2: POST /v1/itineraries** (C3, implements I4)
- Inputs: `destination_id` (string, required), `start_date`/`end_date` (date, required, end_date ≥ start_date), `visibility` (enum: public/private, required), `review` (string, optional), `days` (array of {date, costs: [{category, amount, currency}]}, required)
- Output (201): `{ saved_id: string, status: "saved" }`
- Errors: `422 VALIDATION_ERROR` with field_errors for bad dates/negative costs; `401` if session is invalid; `503` if currency conversion or storage fails (draft is preserved client-side)

**Versioning:** All endpoints are versioned via a `/v1/` URL prefix. Adding an optional field is non-breaking; removing a field, changing a field's type, or changing an existing enum value is a breaking change requiring a new version prefix.

## Technology Choices with Justification

**Database — PostgreSQL:** Team skill fit — standard SQL both teammates have used before, so no ramp-up time. Licensing — PostgreSQL License, permissive and free. Community support — one of the most widely documented open-source databases. Performance — handles the indexed joins and grouped aggregation UC-01 needs at this scale without tuning. Cost and hosting — free tiers exist on low-cost hosts (Render, Railway, Supabase), fitting a no-budget team. *Alternative passed over:* MongoDB was considered but rejected per decision D02, since itinerary/day/cost records are relationally linked and need transactional writes.

**Backend — Node.js with Express:** Team skill fit — JavaScript carries over to the frontend, letting a two-person team share one language. Licensing — MIT. Community support — large ecosystem for the pieces being reused (ORM, auth middleware, date handling). Performance — non-blocking I/O suits an API layer mostly waiting on database and external provider calls. Cost and hosting — deploys cheaply on free/low-cost tiers. *Alternative passed over:* Django was considered for its built-in admin tooling, but Node keeps one language across the stack for a two-person team.

**Frontend — React:** Team skill fit — component-based JS is familiar from coursework. Licensing — MIT. Community support — largest ecosystem for common UI patterns like feeds and filters. Performance — component-level re-rendering is sufficient; Voya has no high-frequency real-time updates. Cost and hosting — static builds deploy free on Vercel/Netlify.

**Hosting — Render (or Railway):** Team skill fit — minimal DevOps knowledge needed, fitting a two-person team with no dedicated infra time. Licensing — not applicable (hosting service). Community support — many tutorials specifically for deploying Node+Postgres apps. Performance — adequate for the estimated scale without manual server management. Cost and hosting — free/near-free starter tier, matching D0's decision to prioritize low-cost shared hosting. *Alternative passed over:* self-managed AWS EC2 was considered but rejected — it needs more configuration time than the team can justify alongside coursework.
