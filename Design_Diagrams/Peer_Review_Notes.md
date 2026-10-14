# Voya: Peer Design Review Notes

Assignment #4 | Tessneem Khalil and Salwa Syed | October 10, 2026

## Feedback and action items

| Area | Feedback | What we changed | Owner | Priority | Status |
|---|---|---|---|---|---|
| Completeness | The season search needs to explain where its weather data comes from. | Added the ClimateNormal model, weekly refresh, and fallback in C5. | Salwa | Must fix | Design updated |
| Clarity | Some interface numbers were being reused for different things. | Kept D0's I1-I13 IDs and listed HTTP routes separately. | Salwa | Must fix | Design updated |
| Completeness | The web client and API layer need their own behavior and errors. | Added C1/C2 responsibilities, handlers, and edge cases. | Tessneem | Must fix | Design updated |
| Feasibility | A remixed trip is supposed to be editable, not a fixed copy. | Kept remixed plans private and editable before publishing. | Tessneem | Must fix | Design updated |
| Risk | Seasonal search should still work if the weather provider fails. | Added stored-data fallback and a warning for older climate data. | Salwa | Must fix | Design updated |
| Traceability | Check requirements against the real user stories. | Verified US-01 to US-05, UC-01, AC-01.1, and AC-01.2. | Both | Must fix | Design updated |

## Review summary

The biggest concerns in missing component details, unclear interfaces, and failure handling. We updated D2 to address those design issues. The requirement table now uses the actual User_Stories document. The three-second response target still needs to be tested when we build the app.

## Two example issues to raise when reviewing other teams

1. What happens when a user saves something and the database or outside provider stops responding? The diagram should show whether anything was saved and how the user can retry.
2. Can each main feature be traced to a user story or acceptance criterion? If a feature does not connect to a requirement, the team should explain why it is included.