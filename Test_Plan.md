# Voya — Test Plan
Team: Tessneem Khalil and Salwa Syed | Advisor: Dr. Jillian Aurisano | Test Lead: Salwa Syed
Traces to: Requirements_Document.md v1.0 (2026-10-29), Technical_Specification.md v1.0 (2026-10-29)

**TC renumbering note:** This plan adds TC-034 through TC-055 to the provisional IDs TC-001–TC-033 already assigned in the Technical Specification's traceability matrix (section 6). No existing TC ID is reused or reassigned. Paste the line below into the Technical Specification's version log (section 1) and update the "Test Case" column of its traceability matrix to list every TC ID now attached to each requirement (see the per-requirement groupings in Part 4 below):

> 1.1 | 2026-11-04 | Test Plan (Week 10) fixes the provisional test-case IDs and adds TC-034 to TC-055, three or more cases per Must requirement. | Salwa Syed, Tessneem Khalil

## Part 1 — Scope

**Tested this cycle, by requirement ID and component:**

| Component | Requirement IDs covered |
|---|---|
| C1 Web Client | REQ-14, REQ-15, NFR-06, NFR-07 |
| C2 API Layer | NFR-04, NFR-05, NFR-10, CON-03, CON-04 |
| C3 Itinerary Service | REQ-04 to REQ-08, REQ-15, NFR-03, NFR-05, NFR-08, CON-05 |
| C4 Cost Aggregation Engine | REQ-01, REQ-02, REQ-03, NFR-01 |
| C5 Season Search Service | REQ-09, REQ-10, NFR-02, NFR-09 |
| C6 Feedback Console | REQ-11, REQ-12 |
| C7 Data Store | REQ-04, REQ-05, NFR-08, CON-07 |
| Auth/Session (glue) | REQ-13, NFR-04, CON-04 |
| Configuration / Logging (glue) | NFR-04, NFR-10, CON-01 |

All 15 functional requirements, all 10 non-functional requirements, and all 8 constraints from Requirements_Document.md are in scope for this cycle.

**Out of scope this cycle:**

- Everything on the Won't list (Requirements_Document.md section 7): notifications, general moderation, password accounts, booking/payments, location/maps, native apps, multiple reviews or review search, in-place editing of public itineraries, precomputed aggregates, following/feeds.
- Third-party components we do not retest: X1 (OIDC provider), X2 (climate API), X3 (exchange-rate API) internal correctness. We test only our handling of their responses (success, timeout, bad data, outage), using stubs, per CON-08.
- Known limitations carried into this cycle: season-rating thresholds are unvalidated against real climate data (D2 gap G3); the 3-second and 2-second performance targets are team assumptions, not instructor-set (A4); NFR-06 usability thresholds are our own targets (A5).
- REQ-12 (reviewer triage) and NFR-10 (request IDs/log scan) are Could-priority; they get one test case each in this cycle rather than the three-case minimum reserved for Must requirements.

## Part 2 — Verification and Validation Strategy

Verification asks whether Voya matches the Requirements and Technical Specification; the team performs it ourselves, against seeded data and stubbed external services, before anything reaches a person. This covers every REQ, NFR, and CON row in the traceability matrix — all of Part 4 below is verification.

Validation asks whether Voya is useful to the stakeholders named in the Week 4 stakeholder map. We validate two ways: a 5-person moderated usability session against NFR-06 (budget-conscious trip planners and itinerary posters, our primary stakeholders, attempting the cost-breakdown and save flows cold), and advisor walkthroughs with Dr. Aurisano at milestones M8, M9, and M12 (Task List, Assignment #8) where she checks a live demo against UC-01 and the US-01–US-05 user stories rather than against the written spec. Any gap the advisor finds between what we built and what a stakeholder actually needs is logged as a change in Requirements_Document.md section 1, not silently patched.

**Test pyramid:**

| Layer | Approx. count | Coverage focus |
|---|---|---|
| Unit | ~40 | Pure logic: reliability-label thresholds, findNearestMonth, season-rating rule, amount/date boundary validation, idempotency-key matching |
| Integration | ~20 | In-process calls between C2 and C3–C6 (I4–I7), C7 transactions (I8, I10, I11), stubbed X1/X2/X3 boundaries (I3, I12, I13) |
| System / end-to-end | 47 | The TC-001–TC-055 cases in Part 4, run against the staging environment through the real HTTP API |
| Acceptance / usability | 6 | NFR-06's 5-person session plus one advisor walkthrough of UC-01 end to end |

Voya has no hardware, sensors, or ML component, so no variables/tolerance/trials table applies.

## Part 3 — Success Criteria Metrics

| Metric | Target threshold | How it is measured | Req ID |
|---|---|---|---|
| Cost-breakdown latency (p95) | ≤ 3 s, 2,000 itineraries seeded, 10 concurrent requests | Load test (k6) against `GET /cost-breakdown` | NFR-01 |
| Season-search latency (p95) | ≤ 3 s, same load as NFR-01 | Load test (k6) against `GET /season-search` | NFR-02 |
| Save latency (p95) | ≤ 2 s, ≤30 days/100 cost items, rates pre-cached | Load test (k6) against `POST /itineraries` with a warmed rate cache | NFR-03 |
| Unauthenticated write success rate | 0 of N attempts succeed (401 every time) | Scripted requests with no session cookie against every write endpoint | NFR-04 |
| Secrets in client code or logs | 0 occurrences | Static scan of the built client bundle; regex scan of log output | NFR-04, NFR-10 |
| Private-itinerary leak rate | 0 of N attempts return itinerary data to a non-owner (always 404) | Scripted `GET /itineraries/{id}` as a different signed-in user | NFR-05 |
| Usability: find breakdown | ≥ 4 of 5 first-time users, ≤ 2 minutes, no help | Moderated session, 5 participants | NFR-06 |
| Usability: save a 3-day trip | ≥ 4 of 5 first-time users, ≤ 10 minutes, no help | Moderated session, 5 participants | NFR-06 |
| Keyboard/label pass rate | 100% of controls in trip-entry, search, breakdown, cost-flag views | Manual keyboard-only walkthrough checklist | NFR-07 |
| Partial-write rate after failure | 0 partial itineraries across 20 injected failures (save, publish, remix) | Fault-injection harness + row-count audit of C7 before/after each injection | NFR-08 |
| Degraded-service correctness | Season search still returns stored normals with an age warning; breakdown and itinerary viewing unaffected, across a 24 h simulated X2 outage | Stubbed-outage harness + response inspection | NFR-09 |
| Reliability-label accuracy | 100% match to n thresholds (reliable n≥3, limited_data n=1–2, no_data n=0) | Assertions against seeded itinerary counts | REQ-01, 02, 03 |
| All-or-nothing save correctness | 0 partial itineraries across boundary and malformed inputs | Boundary-value tests + row audit | REQ-04 |
| Currency-conversion correctness | 100% of saved cost items carry a non-null `amount_usd`, original amount, original currency, and rate date | Assertions against saved rows | REQ-05 |

## Part 4 — Test Cases

Grouped by requirement. Every Must requirement (REQ-01–05, REQ-09, REQ-13, NFR-01, NFR-04, NFR-05, NFR-08) has at least 3 cases including one edge or failure case; every other requirement has at least 1.

| ID | Description | Steps | Expected Result | Status |
|---|---|---|---|---|
| TC-001 | REQ-01 main: breakdown with n≥3 | 1. Seed 5 completed itineraries for `lisbon`/month 6 with cost items in 3 categories. 2. `GET /cost-breakdown?destination_id=lisbon&month=6`. 3. Inspect response. | 200; `average_total_usd` set; each present category shows average/min/max; `n`=5; `reliability`="reliable". | Not Run |
| TC-034 | REQ-01 edge: n=3 boundary | 1. Seed exactly 3 completed itineraries for `porto`/month 4. 2. `GET /cost-breakdown?destination_id=porto&month=4`. 3. Check `n` and category math by hand against seeded values. | `n`=3, `reliability`="reliable" (not limited); averages match hand-computed values exactly. | Not Run |
| TC-035 | REQ-01 failure: C7 unreachable during aggregation | 1. Seed valid data for a destination/month. 2. Block the database connection (kill the pool or point at an invalid host). 3. `GET /cost-breakdown` for that destination/month. | 503 `SERVICE_UNAVAILABLE`; no partial or malformed JSON body. | Not Run |
| TC-002 | REQ-02 main: reliable label with n≥3 | 1. Reuse TC-001 seed data. 2. `GET /cost-breakdown` for that destination/month. 3. Read `n` and `reliability` fields. | `reliability`="reliable" exactly when `n`≥3. | Not Run |
| TC-036 | REQ-02 edge: n=2 limited-data wording | 1. Seed exactly 2 completed itineraries for `rome`/month 9. 2. `GET /cost-breakdown?destination_id=rome&month=9`. 3. Read the client-rendered label. | `reliability`="limited_data"; C1 displays the exact text "Limited data (based on 2 trips)". | Not Run |
| TC-037 | REQ-02 edge: n=2 to n=3 transition | 1. Start with 2 seeded itineraries for `madrid`/month 5 (limited). 2. Add a 3rd completed itinerary for the same destination/month. 3. `GET /cost-breakdown` again. | Label flips from "limited_data" to "reliable" with no other response shape change. | Not Run |
| TC-003 | REQ-03 main: n=0 with an alternate month available | 1. Seed 0 itineraries for `osaka`/month 2, and ≥1 for `osaka`/month 3. 2. `GET /cost-breakdown?destination_id=osaka&month=2`. 3. Read response and client message. | `reliability`="no_data"; `nearest_month`=3; client shows "No cost data yet for this destination in [month]" and suggests month 3; no empty/zero breakdown is rendered. | Not Run |
| TC-038 | REQ-03 edge: no month anywhere has data | 1. Seed 0 itineraries for every month of a fresh destination `reykjavik`. 2. `GET /cost-breakdown?destination_id=reykjavik&month=7`. 3. Read response. | `reliability`="no_data"; `nearest_month`=null; client states plainly that no month has data yet, not a blank screen. | Not Run |
| TC-039 | REQ-03 failure: nearest-month lookup fails | 1. Seed data so a nearest-month lookup is required (as in TC-003). 2. Force the nearest-month query to error (e.g., invalid index state in a test DB). 3. `GET /cost-breakdown` for the no-data month. | 503, not a 200 with an empty or zero-value breakdown. | Not Run |
| TC-004 (T7) | REQ-04 main: save a multi-day, multi-cost itinerary | 1. Sign in. 2. `POST /itineraries` with 3 days, 2 cost items each, mixed categories and currencies. 3. `GET /itineraries/{saved_id}`. | 201 with `saved_id`; the fetched itinerary has exactly the submitted days and cost items, all-or-nothing. | Not Run |
| TC-040 | REQ-04 edge: trip-length and amount boundaries | 1. `POST /itineraries` with end_date exactly 365 days after start_date — expect success. 2. Repeat with 366 days — expect 422. 3. Repeat a valid trip with one cost amount of -1 and another of 1,000,001 — expect 422 with `field_errors` on each. | 365 days saves; 366 days rejected; negative and over-limit amounts both rejected with field-level errors; nothing partially written in any rejected case. | Not Run |
| TC-041 | REQ-04 failure: commit fails mid-transaction | 1. `POST /itineraries` with a valid multi-day draft. 2. Force the C7 commit step to fail after days/cost items are staged (fault injection). 3. Check the database and the response. | 503; 0 rows exist for this itinerary, its days, or its cost items (E-C3-07, NFR-08). | Not Run |
| TC-005 | REQ-05 main: USD conversion stored correctly | 1. `POST /itineraries` with one cost item of 100.00 EUR on a known day. 2. Fetch the saved itinerary. 3. Compare `amount_usd` to the day's rate × 100.00. | `amount_usd` matches the expected conversion; `original_amount`=100.00, `original_currency`="EUR", `rate_date` equals the day's date. | Not Run |
| TC-042 | REQ-05 edge: different days use different rate dates | 1. `POST /itineraries` with a 2-day trip, one cost item on each day, both in GBP. 2. Fetch the saved itinerary. 3. Compare each cost item's `rate_date` to its own day's date. | Each cost item's `rate_date` equals its own day's date, not the trip's start date or save date. | Not Run |
| TC-043 | REQ-05 failure: exchange-rate service exhausts retries | 1. Stub X3 to time out on every call. 2. `POST /itineraries` with a cost item in an uncached currency/date. 3. Check response and database. | After 1 try + 2 retries (3 total), 503; 0 rows written for the itinerary, days, or cost items. | Not Run |
| TC-009 | REQ-09 main: season search by month | 1. Seed `climate_normal` rows for 3 destinations for month 6. 2. `GET /season-search?month=6`. 3. Inspect response. | 200 with all 3 destinations, each carrying `season_rating`, `temp_c`, `rain_days`, `data_as_of`. | Not Run |
| TC-044 | REQ-09 edge: stale climate normal | 1. Seed a `climate_normal` row with `fetched_at` 20 days old. 2. `GET /season-search?month=` that row's month. 3. Inspect the destination's entry. | Destination is still returned, with `stale_warning` true (age > 14 days per REQ-09). | Not Run |
| TC-045 | REQ-09 failure: invalid month | 1. `GET /season-search?month=0`. 2. `GET /season-search?month=13`. | Both return 422 `VALIDATION_ERROR`, not a 200 with empty or wrapped-around data. | Not Run |
| TC-013 | REQ-13 main: sign in through OIDC | 1. `GET /auth/login`, follow the redirect to a stubbed X1. 2. Complete the stub's consent, land on `/auth/callback`. 3. Inspect the session cookie and a subsequent authenticated request. | 302 to X1 with state + PKCE challenge; callback sets a session cookie valid 8 h; `user.oidc_subject` is stored; no password field anywhere. | Not Run |
| TC-046 | REQ-13 edge: cookie at exact 8 h expiry | 1. Sign in and note the issue time. 2. Fast-forward (or wait) to exactly 8 h + 1 s. 3. Make an authenticated request. | 401; the user must sign in again (E-AU-01: `exp <= now` counts as expired). | Not Run |
| TC-047 | REQ-13 failure: tampered callback | 1. `GET /auth/callback` with a `state` value that does not match the one issued at `/auth/login`. 2. Repeat with a well-formed but invalid signature on the callback request. | Both return 400 or 401; no session cookie is set in either case (E-AU-03). | Not Run |
| TC-016 (T16) | NFR-01 main: breakdown latency at load | 1. Seed 2,000 synthetic itineraries across destinations/months (A3). 2. Run a k6 script issuing `GET /cost-breakdown` at 10 concurrent virtual users for 5 minutes. 3. Read the p95 from the k6 report. | p95 response time ≤ 3 s. | Not Run |
| TC-048 | NFR-01 edge: no-data destination under the same load | 1. With the 2,000-itinerary seed in place, run the same 10-concurrent-user load but targeting a destination/month with n=0. 2. Read p95. | No-data responses are not slower than reliable ones; p95 ≤ 3 s. | Not Run |
| TC-049 | NFR-01 failure: connection pool exhaustion | 1. Reduce the test database's connection pool to a small number. 2. Run the load test at 10 concurrent users. 3. Observe response codes and timing. | Requests that cannot get a connection return 503 promptly (bounded by the pool's own timeout), not an unbounded hang. | Not Run |
| TC-019 | NFR-04 main: unauthenticated writes are rejected | 1. For every write endpoint (`POST /itineraries`, `PATCH`/`DELETE /itineraries/{id}`, `POST /itineraries/{id}/remix`, `POST /cost-flags`, `PATCH /admin/cost-flags/{id}`), send the request with no session cookie. | 0 of N attempts succeed; every one returns 401. | Not Run |
| TC-050 | NFR-04 edge: session cookie attributes | 1. Sign in. 2. Inspect the Set-Cookie header from `/auth/callback`. | Cookie is marked HttpOnly and SameSite=Lax and is signed (tamper-evident); no plaintext session data is readable from the client. | Not Run |
| TC-051 | NFR-04 failure/inspection: secret leakage scan | 1. Build the production client bundle. 2. Run a secret-pattern scan (regex for keys, tokens, passwords) over the bundle. 3. Run the same scan over a sample of server log output from a full test run. | 0 matches in the client bundle; 0 matches in the log sample. | Not Run |
| TC-020 | NFR-05 main: private itinerary hidden from non-owner | 1. User A saves a private itinerary. 2. User B (signed in, different account) requests `GET /itineraries/{A's id}`. | 404, not 403, in 0 of N attempts does B see the data (so existence isn't leaked either). | Not Run |
| TC-052 | NFR-05 edge: owner can still read their own private itinerary | 1. User A saves a private itinerary. 2. User A requests `GET /itineraries/{their id}`. | 200 with full itinerary data — confirms the rule blocks only non-owners. | Not Run |
| TC-053 | NFR-05 failure/inspection: no identity leakage on public itineraries | 1. Seed a public itinerary. 2. As an anonymous visitor, `GET /itineraries/{id}`. 3. Inspect the full response body. | 0 occurrences of an email address or login identifier anywhere in the response. | Not Run |
| TC-023 | NFR-08 main: no partial itineraries after 20 injected failures | 1. Run the fault-injection harness across save, publish, and remix flows with 20 induced failures (rate-service down, commit failure, timeout). 2. Audit C7 after each. | 0 of 20 leave a partial itinerary (an itinerary, day, or cost-item row with no matching parent/children). | Not Run |
| TC-054 | NFR-08 edge: failure injected exactly at commit | 1. Stage a valid multi-day save so days and cost items are fully validated and ready to commit. 2. Force the commit statement itself to fail. 3. Count rows for this itinerary before and after. | Row count is identical before and after (0 rows in both cases); client receives 503 and keeps the draft (REQ-14). | Not Run |
| TC-055 | NFR-08 failure: repeated failure on remix | 1. Remix a public itinerary 5 times, injecting a failure on each attempt (commit failure, timeout, rate-service down). 2. After each attempt, fetch the source itinerary. | The source itinerary's days and cost items are byte-for-byte unchanged after all 5 failed attempts; 0 new itineraries are created by a failed remix. | Not Run |
| TC-006 | REQ-06: public/private switch and delete | 1. Owner `PATCH`es an itinerary's `visibility` from private to public, then back. 2. A non-owner confirms it's visible only while public. 3. Owner `DELETE`s it. | Visibility toggles correctly each time; non-owner sees it only while public; delete returns 204 and a subsequent `GET` returns 404. | Not Run |
| TC-007 | REQ-07: edit while private, publish with revalidation | 1. While private, owner edits days/costs/review — succeeds. 2. Owner publishes (sets public) — succeeds, costs reconverted in one transaction. 3. While public, owner attempts to edit days — rejected. | Private edits succeed; publish revalidates and reconverts changed costs atomically; editing days while public returns 409 `IMMUTABLE_WHEN_PUBLIC`. | Not Run |
| TC-008 (T7) | REQ-08: remix into an editable private copy | 1. User B remixes User A's public itinerary. 2. Inspect the new itinerary and the original. | 201 with a new `saved_id` owned by B; all days/cost items copied; review not copied; original itinerary A is byte-for-byte unchanged. | Not Run |
| TC-010 | REQ-10: weekly climate refresh with fallback | 1. Trigger the scheduled refresh with a stubbed X2 returning valid data — rows upsert. 2. Trigger again with X2 returning an error — prior rows remain, retry scheduled for next run. | Valid refresh upserts rows in one transaction; failed refresh keeps all prior valid rows unchanged. | Not Run |
| TC-011 | REQ-11: submit a cost flag | 1. Signed-in user submits a flag for a breakdown key with a 1–2,000 character reason. 2. Submit an 11th flag within the same hour from the same user. | First flags succeed (201, status "new"); the flag beyond the 10-per-hour limit returns 429 `RATE_LIMITED`. | Not Run |
| TC-012 (T7) | REQ-12: reviewer triage, concurrent update | 1. Reviewer lists flags, marks one reviewed with a resolution note. 2. Two reviewers `PATCH` the same flag concurrently using a stale `expected_status`. | First update succeeds (200); the second, using the now-stale `expected_status`, returns 409; the displayed cost figures are unchanged by either. | Not Run |
| TC-014 | REQ-14: draft kept after failed save | 1. Fill in a trip-entry form in C1. 2. Force the save to fail (401, 422, and 503 in three separate attempts). | After each failure type, every field the user entered is still present in the form; nothing needs to be re-typed. | Not Run |
| TC-015 | REQ-15: visibility-correct display | 1. `GET /itineraries/{id}` for a public itinerary as an anonymous visitor — succeeds. 2. Repeat for a private itinerary as a non-owner — 404. | Public itinerary fully visible to anyone; private itinerary visible only to its owner. | Not Run |
| TC-017 (T16) | NFR-02: season-search latency at load | 1. With stored climate normals seeded, run a k6 script at 10 concurrent users against `GET /season-search`. 2. Read p95. | p95 ≤ 3 s. | Not Run |
| TC-018 (T16) | NFR-03: save latency with cached rates | 1. Pre-warm the rate cache for the currencies/dates used. 2. `POST /itineraries` with 30 days and 100 cost items, at 10 concurrent users. 3. Read p95, excluding any run that hits an uncached rate. | p95 ≤ 2 s. | Not Run |
| TC-021 | NFR-06: first-time usability | 1. Recruit 5 participants unfamiliar with Voya. 2. Ask each to find the cost breakdown for a named destination/month, then save a 3-day itinerary, unaided. 3. Time each task. | ≥4 of 5 find the breakdown in ≤2 minutes; ≥4 of 5 save the itinerary in ≤10 minutes. | Not Run |
| TC-022 | NFR-07: keyboard and label accessibility | 1. Using only a keyboard, tab through every control in the trip-entry, search, breakdown, and cost-flag views. 2. Confirm each control has a visible label and a focus state. 3. Confirm reliability is shown as text, not color alone. | 100% of controls are keyboard-reachable with a text label; reliability text equivalents are present. | Not Run |
| TC-024 | NFR-09: degraded service during external outage | 1. Stub X2 to fail for a simulated 24 h. 2. Call `GET /season-search` — expect stored normals with an age warning. 3. Call `GET /cost-breakdown` and `GET /itineraries/{id}` — expect both unaffected. | Season search still returns data with a warning; breakdown and itinerary viewing behave exactly as without the outage. | Not Run |
| TC-025 | NFR-10: request ID and log-secret scan | 1. Make 10 varied API requests. 2. Collect the server logs for that run. 3. Confirm each response carries a request ID and scan the logs for cookies, tokens, or secrets. | Every response has a request ID; 0 secrets found in the log scan. | Not Run |

## Part 5 — Test Environment

| Environment | Purpose | Configuration |
|---|---|---|
| Development | Day-to-day coding and unit/integration tests | Local machine, Node.js LTS, PostgreSQL (local or Docker), stubbed X1/X2/X3, no real network calls to external providers |
| Staging | System and acceptance testing (Part 4's TC cases, load tests) | Deployed server on the free/low-cost Render (or Railway) tier, PostgreSQL on the same tier, static client on Vercel or Netlify preview deploy, real X1/X3 sandbox or recorded-response stubs for X2/X3, seeded with ~2,000 synthetic itineraries (A3) |
| Pilot | NFR-06 usability session and advisor demo (M8, M9, M12) | Same deployment as staging, reset to a clean seeded dataset before each session, demo script fixed in advance |

**Browsers/devices:** latest stable Chrome and Firefox on desktop (primary, since C1 is a responsive React client, CON-03); latest stable Safari on iOS as a secondary check for the keyboard/accessibility cases (NFR-07). No native app, custom hardware, or sensor is in scope (CON-03).

**Network conditions:** staging and pilot tests run over normal broadband; NFR-01/02/03 load tests run from a single test runner against the deployed staging server (not localhost), so latency reflects real network round-trips, not just server processing time.

**Seeded test data:** ~2,000 synthetic itineraries spread across destinations and months for NFR-01/02 load tests (A3); hand-picked small seeds (3–5 itineraries) for REQ-01/02/03 boundary cases so expected averages can be computed by hand; climate-normal rows with both fresh and 15+-day-old `fetched_at` values for REQ-09/REQ-10 staleness cases.

## Part 6 — Pass and Fail Criteria

Each test case passes when its Expected Result in Part 4 is observed exactly; any deviation — wrong status code, wrong field value, wrong label text, a partial write, or a timing miss against Part 3's thresholds — is a fail. A case that cannot be run to completion (missing seed data, environment down) is logged Blocked, not Fail, and is re-run once the blocker is cleared.

**Plan-level pass condition:** the plan as a whole is considered passed when:
- 100% of test cases tied to a Must requirement (REQ-01–05, REQ-09, REQ-13, NFR-01, NFR-04, NFR-05, NFR-08 — 33 cases) pass.
- At least 90% of test cases tied to a Should requirement pass, with any failure either fixed or explicitly deferred and logged as a withdrawn/changed requirement per Requirements_Document.md section 1.
- All Part 3 metrics meet their stated threshold at p95, not on a single best-case run.
- 0 partial-write and 0 private-data-leak cases fail, with no exceptions regardless of priority (NFR-05, NFR-08 are non-negotiable per CON-04/CON-05/CON-07).

**Plan-level fail condition:** any Must-tied test case fails and is not fixed and re-verified before the milestone it gates (Assignment #8 milestones M8–M12), or any metric in Part 3 misses its threshold at the final pre-demo measurement (T16, Week 11–12 of the Task List).

## Part 7 — Milestone, Test Activity, Owner, and Evidence

Placed against the Week 8 Task List and Timeline (Assignment #8).

| Milestone | Test activity | Owner | Evidence |
|---|---|---|---|
| M7 — Foundation live in dev | Unit tests for C7 schema constraints and C2 auth/error mapping; integration test of a live authenticated request | Salwa | Passing unit/integration suite in CI; `GET /healthz` 200 screenshot |
| M8 — Itinerary capture end-to-end | Integration + system tests for REQ-04, REQ-05, REQ-14 (TC-004, 040, 041, 005, 042, 043, 014); unit tests for C3 validation rules | Tessneem | Passing TC run log; sample saved itinerary with converted costs |
| M9 — Remix and cost intelligence working | System tests for REQ-01, 02, 03, 08, 09, 10 (TC-001, 034, 035, 002, 036, 037, 003, 038, 039, 008, 009, 044, 045, 010) | Both (Salwa: C4/C5 cases; Tessneem: remix) | Passing TC run log; hand-verified breakdown math against seeded data |
| M10 — Trust and performance validated | Full NFR-01/02/03 load tests (T16); concurrency tests for REQ-04, REQ-08, REQ-12, NFR-08 (T7) | Salwa (performance), Tessneem (concurrency) | k6 reports showing p95 against Part 3 thresholds; fault-injection audit log showing 0 partial writes |
| M11 — Integration freeze | Full regression of all 47 system-level cases in Part 4; accessibility walkthrough (NFR-07) | Both | Complete Part 4 table with every Status filled in; keyboard-walkthrough checklist |
| M12 — Final demo / symposium | NFR-06 usability session; advisor walkthrough of UC-01 (validation) | Both | Usability session notes and timing data; advisor sign-off note |
| M13 — Design II Week 14 report | Reproduce Part 4 results and Part 3 metrics with actual measured values | Both | Updated Test_Plan.md with final Status column and measured metrics |

## Part 8 — Definition of Done

**Feature-level — a feature is done when:**
- Its code is merged to the main branch and its owner's teammate has reviewed the pull request.
- Every test case tied to its requirement ID in Part 4 has a Status of Pass (or an explicitly logged, advisor-acknowledged deferral).
- Any Part 3 metric tied to its requirement ID meets its threshold on the staging environment, not just locally.
- Its requirement's row in the Technical Specification's traceability matrix (section 6) is updated from "Designed" to "Built" with the final TC IDs.
- No open defect against it is rated higher than low severity.
- Relevant documentation (API contract, data model, or README) reflects the as-built behavior, not just the original design.

**Project-level — Voya is done for this cycle when:**
- All 11 Must requirements meet the plan-level pass condition in Part 6.
- Tests: the full Part 4 table has every case run at least once, with 0 cases left Not Run and 0 left Blocked.
- Review: every merged pull request for the 16 Task List items (T1–T16) has been reviewed by the non-owner partner.
- Documentation: Requirements_Document.md, Technical_Specification.md, and this Test_Plan.md are all at a version consistent with the as-built system, with every change logged in each file's version log.
- Open defects: 0 open defects tied to a Must requirement; any open defect tied to a Should or Could requirement is listed with a severity and an owner.
- Sign-off: Dr. Jillian Aurisano has observed a live demo against UC-01 and confirmed, in the advisor review log (Technical_Specification.md section 7 / Requirements_Document.md Q2), that the built system matches the agreed scope.
