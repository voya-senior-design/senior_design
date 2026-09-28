# User Stories, Use Cases, and Acceptance Criteria - Voya
Team: Salwa Syed, Tessneem Khalil

## Stakeholder Map
- Primary: Budget-conscious trip planner, Itinerary poster
- Secondary: Follower/remixer of another traveler's itinerary
- Hidden: The person handling user complaints about inaccurate cost data, even though Voya has no formal support team yet

## User Stories

**US-01** (Primary - budget trip planner)
As a budget-conscious trip planner,
I want to see a transparent total cost breakdown built from real travelers' reported spending for a destination during a specific month,
so that I can commit to a trip without hidden costs surprising me later.

**US-02** (Primary - itinerary poster)
As a returning traveler,
I want to quickly log my day-by-day itinerary and daily costs while the trip is still fresh,
so that I don't lose the details and can accurately answer "how much did it cost" when people ask later.

**US-03** (Secondary - follower)
As a follower of a traveler whose travel style matches mine,
I want to remix their posted itinerary into my own editable trip plan,
so that I can save planning time while adapting it to my own budget and dates.

**US-04** (Primary - budget trip planner)
As a budget-conscious trip planner,
I want to search destinations by the month I'm free to travel and see which ones have favorable weather and season, so that I only spend time evaluating trips that would actually be enjoyable during my available window.

**US-05** (Hidden - customer support)
As the person handling user complaints about inaccurate cost breakdowns,
I want each cost breakdown to show how many itineraries it's based on,
so that I can explain why an estimate might be off instead of treating every complaint as a bug.

## Use Cases

### UC-01: View Transparent Cost Breakdown for a Destination
Expands: US-01

**Primary actor:** Budget-conscious trip planner
**Secondary actors:** None

**Preconditions:**
- The planner has selected a destination and a target month.
- At least one prior traveler has submitted a completed itinerary with cost data for that destination within the past 24 months.

**Main success flow:**
1. Planner selects a destination and a travel month.
2. System retrieves all itineraries tagged with that destination and month.
3. Planner requests the cost breakdown.
4. System aggregates reported costs by category (flights, lodging, food, activities) across matching itineraries and displays an average total plus a per-category range.
5. Planner reviews the breakdown.
6. System displays the number of itineraries the breakdown is based on, so the planner can judge how reliable the estimate is.

**Alternate flow (partial data):**
4a. If fewer than 3 itineraries match the destination and month, system still displays the aggregate but flags it as "Limited data (based on 1–2 trips)" instead of presenting it as a reliable average.

**Exception flow (no data):**
2a. If zero itineraries match the destination and month, system displays "No cost data yet for this destination in [month]" and suggests the nearest month with available data instead of showing an empty or broken breakdown.

**Postcondition:** The planner has either a cost estimate (reliable or flagged as limited) or a clear explanation of why none exists, and in no case sees a blank or misleading result.

## Acceptance Criteria

**AC-01.1** (main success flow)
Given a destination and month with at least 3 matching itineraries,
When the planner requests the cost breakdown,
Then the system displays a total average cost and a per-category range within 3 seconds, along with the number of itineraries used.

**AC-01.2** (exception flow)
Given a destination and month with zero matching itineraries,
When the planner requests the cost breakdown,
Then the system displays a "no data yet" message and suggests at least one alternate month, and does not display an empty or zero-value breakdown.
