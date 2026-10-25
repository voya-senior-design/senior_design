# Tasklist.md

T1. Specify the C1 Web Client screens and component states for trip entry, search, and the cost-report view. Owner: Tessneem.

T2. Develop the C1 Web Client trip-entry and itinerary-draft forms with field validation and idempotent submit. Owner: Tessneem.

T3. Develop the C3 Itinerary Service save, edit, and publish endpoints per the D2-3 state machine. Owner: Tessneem.

T4. Develop the C3 Itinerary Service remix endpoint with ownership checks and a transactional copy. Owner: Tessneem.

T5. Develop the C6 Feedback Console cost-flag submission and admin review endpoints. Owner: Tessneem.

T6. Develop the C1 Web Client season-search and cost-breakdown display views. Owner: Tessneem.

T7. Test concurrency edge cases E-C3-03, E-C3-04, and E-C6-03 (double submit, remix-during-delete, dual reviewer) from the D2 edge-case table. Owner: Tessneem.

T8. Validate the C1/C2 interface contract against D0's thirteen interfaces and refine 401/422/503 error handling across every C1 view. Owner: Tessneem.

T9. Obtain and configure credentials and rate limits for the X1 OIDC provider, X2 climate-data API, and X3 exchange-rate API. Owner: Salwa.

T10. Develop the C7 Data Store schema and migrations for User, Destination, Itinerary, Day, CostItem, ClimateNormal, and CostFlag. Owner: Salwa.

T11. Develop the C2 API Layer request routing, session authentication, and domain-error-to-status mapping. Owner: Salwa.

T12. Develop the Logging glue module (structured stdout logging, request IDs, secret redaction). Owner: Salwa.

T13. Develop the Auth/Session glue module (OIDC login, callback, logout, signed cookie issuance). Owner: Salwa.

T14. Develop the C4 Cost Aggregation Engine query and reliability-labeling logic (reliable / limited_data / no_data). Owner: Salwa.

T15. Develop the C5 Season Search Service rating rule and weekly climate-normal refresh job. Owner: Salwa.

T16. Validate C4/C5 query performance against the D1 100x data-scale estimate using about 2,000 seeded itineraries, closing gap G1. Owner: Salwa.
