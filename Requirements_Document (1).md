# Voya - Requirements Document

Team: Tessneem Khalil and Salwa Syed | Advisor: Dr. Jillian Aurisano

## 1. Version and Change Log

| Version | Date | Change | Author |
|---|---|---|---|
| 1.0 | 2026-10-29 | Initial requirements draft derived from User_Stories.md and Design D0 to D2, with measurable conditions, verification methods, MoSCoW summary, constraints (CON-##), glossary, and assumptions. | Tessneem Khalil, Salwa Syed |

Rules for this log: IDs are stable and never reused. If a requirement is dropped, it is marked **withdrawn** in its table and logged here. It is not renumbered or deleted.

## 2. Purpose and Scope

**Purpose.** This document says what Voya must do and how well, in terms a teammate can check as pass or fail. The Technical Specification (version 1.0) traces every ID below to a design element and a planned test.

**Product.** Voya is a social platform for travelers to share trip reviews, recommendations, itineraries, and costs in one place. People can see where others traveled during different months and use their experiences to plan a trip that fits their budget.

**In scope for this release.** Sign-in, saving and publishing itineraries with daily costs, viewing and remixing public itineraries, month-specific cost breakdowns with a sample size and reliability label, destination search by month, and cost-accuracy flags with a reviewer view.

**Out of scope.** Everything on the Won't list in section 7.

**Source documents.** User_Stories.md (US-01 to US-05, UC-01, AC-01.1, AC-01.2), and Design D0, D1, D2 in `Design_Diagrams/`.

## 3. Stakeholders and User Goals

From the Week 4 stakeholder map.

| Stakeholder | Type | Goal |
|---|---|---|
| Budget-conscious trip planner | Primary | See a transparent cost breakdown for a destination and month, and find destinations that fit the month they are free (US-01, US-04). |
| Itinerary poster | Primary | Log a day-by-day itinerary and costs while the trip is fresh, so they can answer "how much did it cost" later (US-02). |
| Follower / remixer | Secondary | Copy another traveler's itinerary into an editable plan and adapt it to their own budget and dates (US-03). |
| Person handling complaints about inaccurate costs | Hidden | See how many itineraries each breakdown is based on, so an estimate can be explained instead of treated as a bug (US-05). Voya has no formal support team yet, so this is one of the two team members. |

## 4. Glossary

Each term is defined once here and used with the same meaning in every Voya document.

| Term | Meaning |
|---|---|
| **Planner, Poster, Follower, Reviewer** | The four roles from section 3. A **Planner** reads breakdowns and searches. A **Poster** saves itineraries. A **Follower** remixes someone else's itinerary. A **Reviewer** is the signed-in team member who handles cost flags. One person can hold several roles except Reviewer. |
| **Visitor** | Anyone using Voya who is not signed in. |
| **Itinerary** | One trip: a destination, start and end date, visibility, an optional review and recommendations, and one or more days. |
| **Day / Cost item** | A **day** is one calendar date of an itinerary. A **cost item** is one expense on a day: category, amount, currency. |
| **Category** | flights, lodging, food, activities, transport, or other. |
| **Destination** | A place identified by a slug such as `lisbon`. |
| **Month** | The calendar month (1 to 12) of an itinerary's start date. |
| **Completed itinerary** | A saved itinerary with at least one cost item. Only completed itineraries count toward a breakdown. |
| **Public / Private** | **Public** itineraries can be seen by everyone. **Private** itineraries can be seen only by their owner. |
| **Draft** | Trip details held in the browser and not yet saved. |
| **Remix** | Creating a new private itinerary owned by the user as a copy of an existing one. |
| **Cost breakdown** | For one destination and month: the average total cost in USD and, per category, the average, minimum, and maximum. |
| **n** | The number of distinct completed itineraries a breakdown is based on. |
| **Reliability label** | **reliable** (n of 3 or more), **limited data** (n of 1 or 2), **no data** (n of 0). |
| **Breakdown key** | A destination and month pair that identifies one breakdown. |
| **Cost flag** | A signed-in user's report that a displayed breakdown looks wrong. Status is new, reviewed, or dismissed. |
| **Climate normal** | A stored monthly average temperature (degrees C) and number of rain days for a destination. |
| **Season rating** | good, mixed, or challenging, computed from a climate normal by a configurable rule. |
| **Components C1 to C7** | C1 Web Client, C2 API Layer, C3 Itinerary Service, C4 Cost Aggregation Engine, C5 Season Search Service, C6 Feedback Console, C7 Data Store (as in Design D0). |
| **External services X1 to X3** | X1 identity provider (OpenID Connect), X2 climate data API, X3 exchange-rate API. |
| **MoSCoW** | Must, Should, Could, Won't priority for this release. |

## 5. Functional Requirements

Verification methods: **Test** (automated or scripted check), **Inspection** (read the code, config, or output), **Demonstration** (live walkthrough against a use case).

| ID | Requirement | Priority | Source | Verification |
|---|---|---|---|---|
| REQ-01 | The system shall display, for a destination and month with n of 3 or more, the average total cost in USD and, for each category that has data, the average, minimum, and maximum, using completed itineraries from the last 24 months. | Must | US-01, UC-01, AC-01.1 | Test with seeded itineraries |
| REQ-02 | The system shall show with every cost breakdown the value of n and a reliability label: "reliable" when n is 3 or more, "Limited data (based on n trips)" when n is 1 or 2. | Must | US-05, UC-01 | Test |
| REQ-03 | When n is 0 for the requested destination and month, the system shall display "No cost data yet for this destination in [month]" and suggest at least one other month with n of 1 or more, and shall never display an empty or zero-value breakdown. If no month has data, it shall say that. | Must | AC-01.2, UC-01 | Test |
| REQ-04 | The system shall let a signed-in user save an itinerary (destination, start and end date, visibility, review, recommendations, and per-day cost items) as one all-or-nothing operation, shall reject a trip longer than 365 days or an amount that is negative or above 1,000,000, and shall return a saved ID. | Must | US-02 | Test |
| REQ-05 | The system shall convert each cost item to USD using the exchange rate for the date of the day containing that cost item, and store the original amount, original currency, rate date, and USD amount. If no rate is obtained after 3 attempts (1 try plus 2 retries), the save shall fail with a 503 and store nothing. | Must | US-02, D0 I13 | Test with a stubbed rate service |
| REQ-06 | The system shall let the owner of an itinerary switch it between public and private or delete it, and shall show only public itineraries to anyone but the owner. | Should | US-02, US-03 | Test |
| REQ-07 | While an itinerary is private, the system shall let its owner replace its days, cost items, and review. While it is public, the system shall reject day and cost changes with a 409. Publishing shall revalidate the plan and reconvert changed costs in one transaction, and the itinerary shall stay private if that fails. | Should | US-03 | Test |
| REQ-08 | The system shall let a signed-in user remix any public itinerary (or their own) into a new private itinerary they own, copying all days and cost items but not the review, in one transaction, and leaving the original unchanged. | Should | US-03 | Test |
| REQ-09 | The system shall let a user choose a month and shall list destinations with a season rating (good, mixed, or challenging), temperature, rain days, and a refresh date for each destination, and shall show a warning for any climate normal older than 14 days. | Must | US-04 | Test |
| REQ-10 | The system shall refresh climate normals from X2 once a week, validate them, and keep the previous valid rows if the refresh fails. If no valid normal exists it shall return a 503 and shall never show a guessed climate value. | Should | US-04 | Test with a stubbed climate service |
| REQ-11 | The system shall let a signed-in user submit a cost flag (breakdown key, the n they saw, and a reason of 1 to 2,000 characters), limited to 10 flags per user per hour. | Should | US-05 | Test |
| REQ-12 | The system shall let a Reviewer list cost flags and mark each reviewed or dismissed with a resolution note, shall return a 409 to a second reviewer who updates a flag from a stale status, and shall not change any displayed cost figure because of a flag. | Could | US-05 | Test |
| REQ-13 | The system shall sign users in through an external OpenID Connect provider (X1) and keep them signed in for 8 hours, and shall store no passwords. | Must | US-02, D0 section 6 | Test and Inspection |
| REQ-14 | When a save fails with a 401, 422, or 503, the system shall keep every value the user entered in the browser so the user can retry without re-entering anything. | Should | US-02 | Test |
| REQ-15 | The system shall display a public itinerary (days, cost items, review, recommendations) to any visitor, and a private itinerary only to its owner. | Should | US-02, US-03, D0 section 1 | Test |

## 6. Non-Functional Requirements

| ID | Category | Requirement | Priority | Source | Verification |
|---|---|---|---|---|---|
| NFR-01 | Performance | A cost breakdown request shall complete in 3 seconds or less at the 95th percentile with 2,000 itineraries stored and 10 concurrent requests. | Must | AC-01.1 | Test (load test, task T16) |
| NFR-02 | Performance | A season search request shall complete in 3 seconds or less at the 95th percentile under the same load as NFR-01. | Should | US-04, D0 section 5 | Test (task T16) |
| NFR-03 | Performance | Saving an itinerary of up to 30 days and 100 cost items shall complete in 2 seconds or less at the 95th percentile, when all required dated exchange rates are cached, excluding external provider outage and retry time. | Should | US-02, D0 section 5 | Test |
| NFR-04 | Security | Every write request shall be rejected with a 401 when it has no valid session (0 of the unauthenticated write attempts in the test suite succeed). The session cookie shall be signed, HTTP-only, and SameSite=Lax, and no secret or password shall appear in client code or logs. | Must | D0 section 6 | Test and Inspection |
| NFR-05 | Security | A private itinerary shall be returned to a non-owner in 0 of the test attempts (response is a 404), and no public itinerary response shall contain an email address or login identifier. | Must | D0 section 6 | Test |
| NFR-06 | Usability | At least 4 of 5 first-time users shall find the cost breakdown for a named destination and month within 2 minutes without help, and at least 4 of 5 shall save a 3-day itinerary within 10 minutes. | Should | US-01, US-02 | Demonstration (5-person session) |
| NFR-07 | Usability (accessibility) | Every interactive control in the trip-entry, search, breakdown, and cost-flag views shall be reachable by keyboard and have a text label, and reliability shall be shown in text as well as color (100 percent of controls pass a keyboard walkthrough). | Should | D0 section 6 | Inspection |
| NFR-08 | Reliability | Across 20 injected failures during save, publish, and remix (rate service down, commit failure, timeout), 0 partial itineraries shall remain in the data store. | Must | US-02, D02 | Test |
| NFR-09 | Reliability | With the climate service unavailable for 24 hours, season search shall still return results from stored normals with an age warning, and the cost breakdown and itinerary viewing shall be unaffected. | Should | US-04, D0 D04 | Test |
| NFR-10 | Reliability (observability) | Every API response shall carry a request ID, and a scan of the logs from the test run shall find 0 cookies, tokens, or secrets. | Could | D0 section 6 | Inspection |

## 7. MoSCoW Summary

Of 25 requirements, 11 are Must (44 percent), so the Must set is a realistic first version: sign in, save a trip, convert currency, see a cost breakdown with its sample size, search by month, and the security and no-partial-save guarantees.

**Must (11):** REQ-01 cost breakdown, REQ-02 n and reliability label, REQ-03 no-data message and alternate month, REQ-04 save itinerary, REQ-05 currency conversion, REQ-09 season search, REQ-13 sign-in, NFR-01 breakdown speed, NFR-04 write authentication, NFR-05 private itineraries stay private, NFR-08 no partial saves.

**Should (12):** REQ-06 public/private switch, REQ-07 edit and publish, REQ-08 remix, REQ-10 climate refresh and fallback, REQ-11 submit cost flag, REQ-14 keep draft after failed save, REQ-15 view itineraries, NFR-02 season search speed, NFR-03 save speed, NFR-06 usability study, NFR-07 accessibility, NFR-09 degraded-service behavior.

**Could (2):** REQ-12 reviewer triage of flags, NFR-10 request IDs and log scan.

**Won't for this release.** These stay out so they cannot creep back in:

- Email or other notifications (the optional outbox in D2 section 2.6 is not built).
- General moderation, reporting, or appeals for itineraries and reviews. Cost flags only.
- Password accounts or any credential storage by Voya.
- Booking, reservations, or payments.
- Location tracking, maps, or routes.
- Native mobile apps. The client is a web page.
- Multiple reviews, comments, voting, or review search on an itinerary.
- Editing the days or costs of a public itinerary in place. The owner makes it private first.
- Precomputed or cached cost aggregates. Breakdowns are computed on read.
- Following other users or personalized feeds.

## 8. Inherited Constraints

These are drawn from D0 section 6 and the project setup. They still need to be checked against the original Week 3 essay, which was not provided. Each is restated with its consequence in the Technical Specification.

| ID | Constraint |
|---|---|
| CON-01 | Cost: use low-cost or free-tier hosting and services. There is no budget for paid infrastructure. |
| CON-02 | Team and time: two people, one implementation semester of 14 weeks (Monday 2027-01-11 to Friday 2027-04-23), with the final demo on 2027-04-21. |
| CON-03 | Platform: users reach Voya through a standard web browser. No custom hardware, sensors, or native apps. |
| CON-04 | Identity: store the identity provider's subject identifier, never passwords, and keep all credentials on the server. |
| CON-05 | Privacy: do not collect location traces or booking and payment details. Public itineraries omit private account details. |
| CON-06 | Accessibility: keyboard navigation, labeled fields, and text equivalents for reliability colors. |
| CON-07 | Architecture: one modular server (C2 to C6) with one relational data store (C7). No separately deployed microservices. |
| CON-08 | External services: X1, X2, and X3 are outside our control. Climate lookups must not require live provider calls, and exchange-rate or sign-in failures must return clear errors without partial writes. |

## 9. Assumptions and Open Questions

| ID | Item | Owner | Date |
|---|---|---|---|
| A1 | The spring term runs Jan 11 to Apr 23, 2027. If it differs, all planned dates shift by the same offset. | Both | 2026-11-13 |
| A2 | Free tiers of the host, X1, X2, and X3 are enough for development, testing, and the demo. | Salwa | 2027-01-15 |
| A3 | About 2,000 synthetic itineraries represent the D1 100x scale estimate well enough for NFR-01 and NFR-02. | Salwa | 2027-04-09 |
| A4 | The 3 second targets (NFR-01, NFR-02) and the 2 second save target (NFR-03) come from D0 as team assumptions and the AC-01.1 acceptance criterion, not from the instructor. | Both | 2026-11-13 |
| A5 | The usability thresholds in NFR-06 (4 of 5 users, 2 and 10 minutes) are our own targets. | Tessneem | 2027-03-27 |
| A6 | The season rating thresholds (good: 15 to 28 degrees C and at most 10 rain days) are our design choice and have not been checked against real climate data (gap G3 in D2). | Salwa | 2027-03-27 |
| Q1 | Confirm CON-01 to CON-08 against the original Week 3 essay before submission. That essay was not included with these files. | Both | 2026-11-02 |
| Q2 | Complete the advisor review after the actual meeting and log any resulting changes in section 1 and Technical Specification section 7. | Both | 2026-11-05 |
