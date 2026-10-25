# Voya — Task List, Milestones, Timeline, and Effort Matrix
Assignment #5 and #6 | Team: Tessneem Khalil and Salwa Syed

**Planning assumption:** Design I (Assignment #4, D0–D2) runs through Fall 2026. The Task List below covers the implementation semester, CS 5002 Senior Design Project II, which we assume runs Monday, January 11 – Friday, April 23, 2027 (14 weeks). If the actual Spring 2027 calendar differs, every date below shifts by the same offset; the durations, dependencies, and buffers do not change.

## Task List (Assignment #5)

Sixteen tasks, about eight per member, each sized to one or two weeks and tied to a D0–D2 module.

**Tessneem Khalil**

1. Specify the C1 Web Client screens and component states for trip entry, search, and the cost-report view. (C1 Web Client) Owner: Tessneem.
2. Develop the C1 Web Client trip-entry and itinerary-draft forms with field validation and idempotent submit. (C1 Web Client) Owner: Tessneem.
3. Develop the C3 Itinerary Service save, edit, and publish endpoints per the D2-3 state machine. (C3 Itinerary Service) Owner: Tessneem.
4. Develop the C3 Itinerary Service remix endpoint with ownership checks and a transactional copy. (C3 Itinerary Service) Owner: Tessneem.
5. Develop the C6 Feedback Console cost-flag submission and admin review endpoints. (C6 Feedback Console) Owner: Tessneem.
6. Develop the C1 Web Client season-search and cost-breakdown display views. (C1 Web Client) Owner: Tessneem.
7. Test concurrency edge cases E-C3-03, E-C3-04, and E-C6-03 (double submit, remix-during-delete, dual reviewer) from the D2 edge-case table. Owner: Tessneem.
8. Validate the C1/C2 interface contract against D0's thirteen interfaces and refine 401/422/503 error handling across every C1 view. Owner: Tessneem.

**Salwa Syed**

9. Obtain and configure credentials and rate limits for the X1 OIDC provider, X2 climate-data API, and X3 exchange-rate API. (Configuration) Owner: Salwa.
10. Develop the C7 Data Store schema and migrations for User, Destination, Itinerary, Day, CostItem, ClimateNormal, and CostFlag. (C7 Data Store) Owner: Salwa.
11. Develop the C2 API Layer request routing, session authentication, and domain-error-to-status mapping. (C2 API Layer) Owner: Salwa.
12. Develop the Logging glue module (structured stdout logging, request IDs, secret redaction). (Logging) Owner: Salwa.
13. Develop the Auth/Session glue module (OIDC login, callback, logout, signed cookie issuance). (Auth/Session) Owner: Salwa.
14. Develop the C4 Cost Aggregation Engine query and reliability-labeling logic (reliable / limited_data / no_data). (C4 Cost Aggregation Engine) Owner: Salwa.
15. Develop the C5 Season Search Service rating rule and weekly climate-normal refresh job. (C5 Season Search Service) Owner: Salwa.
16. Validate C4/C5 query performance against the D1 100x data-scale estimate using about 2,000 seeded itineraries, closing gap G1. (C4 Cost Aggregation Engine / C5 Season Search Service) Owner: Salwa.

*(Saved separately as `Tasklist.md` in the repository root, same numbering and wording.)*

## Milestones List (Assignment #6)

| ID | Milestone | What it demonstrates | Date | Verified by | Owner |
|---|---|---|---|---|---|
| M0 | Advisor recorded and team contract approved | Dr. Jillian Aurisano confirmed as advisor; contract signed | 2026-09 (complete) | Signed contract + spreadsheet entry on Canvas | Both |
| M1 | User Stories, Use Cases, Acceptance Criteria submitted | US-01–US-05, UC-01, AC-01.1/01.2 | 2026-10-07 (complete) | `User_Stories.md` committed, tagged `week04` | Both |
| M2 | D0 High-Level Design submitted | 7 components, 13 interfaces, 2 data-flow diagrams, decision log | 2026-10-02 (complete) | PDF + sources in `Design_Diagrams/` | Both |
| M3 | D1 Detailed Design submitted | C3/C4 data model and aggregation algorithm, API contract, tech stack | 2026-10-07 (complete) | PDF + sources in `Design_Diagrams/` | Both |
| M4 | D2 Detailed Design + peer review submitted | C1/C2/C5/C6 behavior, state machines, edge-case and traceability tables | 2026-10-14 (complete) | PDF + `Peer_Review_Notes.md` | Both |
| M5 | Task List, Milestones, Timeline, Effort Matrix submitted | This document | 2026-10-28 (complete on submission) | PDF + `Tasklist.md` committed | Both |
| M6 | Design I Week 14 report / final presentation | Design phase closed out to advisor and instructor | 2026-12-02 (planned) | Presentation + report on Canvas | Both |
| M7 | Foundation live in dev: C7 schema, C2 API layer, Configuration, Logging | A deployed dev environment accepts an authenticated request end-to-end | 2027-02-11 | Live `GET /healthz` call + manual auth smoke test | Salwa |
| M8 | Authenticated itinerary capture working end-to-end | Trip entry → save → edit → publish, C1 through C3 to C7 | 2027-02-25 | Live demo of the save/edit/publish flow (UC against US-02) | Tessneem |
| M9 | Remix and cost intelligence working | Remix, cost breakdown, and season search return real data | 2027-03-27 | Live demo against UC-01, AC-01.1/01.2, with seeded data | Salwa |
| M10 | Trust and performance validated | Cost flags, concurrency tests, and the G1 performance test all pass | 2027-04-12 | Passing test suite + a written load-test result for T16 | Both |
| M11 | Integration freeze | All sixteen tasks merged and demo script rehearsed | 2027-04-18 | Full walkthrough of every use case against its acceptance criteria | Both |
| M12 | Final demo / capstone symposium | Live, working Voya demo to advisor and instructor | 2027-04-21 | Live demo | Both |
| M13 | Design II Week 14 report | Implementation progress reproduces these tables with actuals | 2027-04-23 | Report submitted on Canvas | Both |

## Table 1 — Timeline

Each task's completion date already includes a 15–20% schedule buffer. A protected five-day block with no new task starts precedes each major milestone (M7, M9, M11, M12).

| Task ID | Statement (short) | Owner | Start | Completion (incl. buffer) | Buffer | Dependencies |
|---|---|---|---|---|---|---|
| T9 | Configuration: obtain API credentials | Salwa | 2027-01-12 | 2027-01-15 | 1 day (~20%) | none |
| T10 | C7 Data Store schema and migrations | Salwa | 2027-01-15 | 2027-01-28 | 2 days (~15%) | T9 |
| T1 | Specify C1 screens and states | Tessneem | 2027-01-12 | 2027-01-23 | 2 days (~15%) | none |
| T11 | C2 API Layer routing + auth mapping | Salwa | 2027-01-28 | 2027-02-11 | 2 days (~15%) | T10, T9 |
| T2 | C1 trip-entry and draft forms | Tessneem | 2027-01-26 | 2027-02-09 | 2 days (~15%) | T1, T11 (contract) |
| — | **Protected block before M7** | Both | 2027-02-08 | 2027-02-11 | — | — |
| **M7** | **Foundation live in dev** | Both | — | **2027-02-11** | — | T9, T10, T11 |
| T12 | Logging glue module | Salwa | 2027-02-11 | 2027-02-13 | 1 day (~20%) | T11 |
| T13 | Auth/Session glue module | Salwa | 2027-02-12 | 2027-02-24 | 2 days (~15%) | T11 |
| T3 | C3 save/edit/publish endpoints | Tessneem | 2027-02-10 | 2027-02-25 | 3 days (~18%) | T10, T11 |
| **M8** | **Itinerary capture end-to-end** | Both | — | **2027-02-25** | — | T3, T13 |
| T4 | C3 remix endpoint | Tessneem | 2027-02-26 | 2027-03-05 | 1 day (~15%) | T3 |
| T14 | C4 Cost Aggregation Engine logic | Salwa | 2027-02-25 | 2027-03-12 | 3 days (~18%) | T11, T10 |
| T5 | C6 Feedback Console endpoints | Tessneem | 2027-03-08 | 2027-03-19 | 2 days (~15%) | T10, T11 |
| T15 | C5 Season Search Service + refresh job | Salwa | 2027-03-15 | 2027-03-27 | 3 days (~20%, spans spring break) | T11, T10 |
| — | **Protected block before M9** | Both | 2027-03-24 | 2027-03-27 | — | — |
| **M9** | **Remix and cost intelligence working** | Both | — | **2027-03-27** | — | T4, T14, T15 |
| T6 | C1 season-search and cost views | Tessneem | 2027-03-22 | 2027-04-02 | 2 days (~15%) | T14, T15 |
| T16 | C4/C5 performance validation (closes G1) | Salwa | 2027-03-29 | 2027-04-09 | 2 days (~15%) | T14, T15, T10 |
| T7 | Concurrency tests E-C3-03/04, E-C6-03 | Tessneem | 2027-04-05 | 2027-04-12 | 1 day (~15%) | T4, T5 |
| **M10** | **Trust and performance validated** | Both | — | **2027-04-12** | — | T7, T16 |
| T8 | C1/C2 interface + error-handling validation | Tessneem | 2027-04-12 | 2027-04-16 | 1 day (~15%) | T6, T7 |
| — | **Protected block before M11/M12** | Both | 2027-04-16 | 2027-04-20 | — | — |
| **M11** | **Integration freeze** | Both | — | **2027-04-18** | — | T8, T16 |
| **M12** | **Final demo / symposium** | Both | — | **2027-04-21** | — | M11 |
| **M13** | **Design II Week 14 report** | Both | — | **2027-04-23** | — | M12 |

## Table 2 — Effort Matrix

Hours estimated with PERT: (Optimistic + 4×Most-likely + Pessimistic) / 6. Each task's primary owner carries the larger share; the partner's hours cover code review and integration testing on that task.

| Task | Tessneem (hrs) | Salwa (hrs) | Total (hrs) | Primary owner |
|---|---|---|---|---|
| T1 | 10 | 2 | 12 | Tessneem |
| T2 | 16 | 3 | 19 | Tessneem |
| T3 | 17 | 3 | 20 | Tessneem |
| T4 | 10 | 2 | 12 | Tessneem |
| T5 | 12 | 2 | 14 | Tessneem |
| T6 | 14 | 3 | 17 | Tessneem |
| T7 | 10 | 2 | 12 | Tessneem |
| T8 | 9 | 2 | 11 | Tessneem |
| T9 | 1 | 6 | 7 | Salwa |
| T10 | 2 | 12 | 14 | Salwa |
| T11 | 3 | 16 | 19 | Salwa |
| T12 | 1 | 6 | 7 | Salwa |
| T13 | 2 | 12 | 14 | Salwa |
| T14 | 3 | 16 | 19 | Salwa |
| T15 | 3 | 14 | 17 | Salwa |
| T16 | 2 | 10 | 12 | Salwa |
| **Column total** | **115** | **111** | **226** | — |

**Balance note:** the columns are within 4 hours of each other (115 vs. 111), so no rebalancing is planned. If either partner's co-op rotation or exam schedule removes them for more than a week (see risk R3), we will shift a same-owner task into the other partner's queue using the support hours already budgeted above, rather than rewriting the matrix.

## Work Breakdown Structure

- **Phase 1 — Core Services Foundation**
  - T9 Configuration
  - T10 C7 Data Store
  - T11 C2 API Layer
  - T12 Logging
  - T13 Auth/Session
- **Phase 2 — Trip Capture and Itinerary Lifecycle**
  - T1 C1 screen specification
  - T2 C1 trip-entry forms
  - T3 C3 save/edit/publish
  - T4 C3 remix
- **Phase 3 — Discovery and Cost Intelligence**
  - T14 C4 Cost Aggregation Engine
  - T15 C5 Season Search Service
  - T6 C1 season-search and cost views
- **Phase 4 — Trust, Quality, and Demo Readiness**
  - T5 C6 Feedback Console
  - T7 Concurrency tests
  - T16 Performance validation (G1)
  - T8 Interface and error-handling validation

## Risk Register

| ID | Category | Risk | Likelihood | Impact | Strategy | Mitigation | Owner | Trigger |
|---|---|---|---|---|---|---|---|---|
| R1 | Technical | Few real users means most destinations sit in `no_data` state, so the demo looks empty | High | Medium | Reduce | Seed ~2,000 synthetic itineraries by T16 (Week 11) so demo destinations are always `reliable` | Salwa | Fewer than 3 itineraries exist for a planned demo destination two weeks before M12 |
| R2 | Technical | X2 (climate) or X3 (exchange-rate) API changes schema or rate limits, breaking C5/C3 | Medium | Medium | Reduce | Cache normals/rates in C7 (D04/D07) with a stored-fallback path so provider downtime degrades to stale data instead of failure | Salwa | Provider returns non-200 in dev/CI for more than 24 hours |
| R3 | Team | A GE Aerospace co-op rotation, exam week, or travel removes one partner for 1–2 weeks | Medium | High | Reduce | Support hours in Table 2 keep the non-owner partner current enough to cover a blocked task for up to a week; flag any known conflict to the advisor 2+ weeks ahead | Both | Either partner announces a conflict overlapping a task's planned window |
| R4 | External | UC spring break falls inside the T15/T5 window (mid-March) and reduces available hours that week | High | Low–Medium | Accept, with buffer | The 15–20% task buffer absorbs a short low-output week; no new task start is scheduled during the break itself | Both | Calendar-certain; reviewed at the M9 check-in |
| R5 | Technical | The 3-second AC-01.1 performance target (open gap G1) fails once C4 runs against realistic data on a free-tier host | Medium | High | Reduce | T16 runs a dedicated load test against ~2,000 seeded itineraries with 3 weeks of buffer before M11 to add an index or caching if missed | Salwa | T16's measured p95 response time exceeds 3 seconds |
| R6 | Team | Scope creep adds an out-of-scope feature (e.g., full moderation, review voting — D2 gap G5) that consumes budgeted time | Medium | Medium | Avoid | Any new feature is checked against the D2 Requirements Traceability Table before being added; untraceable features go to a Phase-2 backlog instead of this plan | Both | A feature is proposed with no US-xx or UC-xx ID behind it |
| R7 | External | The free/low-cost hosting tier sleeps or rate-limits an idle service, causing a cold-start delay right before a graded demo | Medium | Medium | Reduce | Confirm sleep/idle behavior during T11 (Week 4); add a keep-alive ping or upgrade tier before M7 if the service sleeps | Salwa | Dev deployment cold-starts (>5s to first response) after idle |
| R8 | Technical | Stdout-only logging (accepted in D2, E-LG-04) means log data is lost if the host's log pipeline drops output, hiding a late bug's cause | Low | Low | Accept | None beyond the existing design — `GET /healthz` and the database remain the source of truth for debugging | Salwa | Accepted; revisited only if logs are needed for a specific incident |

High-impact schedule risks (R1, R3, R5, R7) are reflected in Table 1: R1 and R5 are why T16 sits three weeks before the integration freeze (M11) rather than immediately before it, R3 is why every task carries partner support hours in Table 2, and R7 is why hosting behavior is checked during T11 in Week 4 rather than discovered at demo time.
