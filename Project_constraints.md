Voya \- Project Constraints Essay  
Team: Salwa Syed, Tessneem Khalil

**Economic**  
Voya is being built by a two-person student team with no real budget, so our economic constraint is choosing tools with generous free tiers rather than allocating funds. This means our weather, mapping, and pricing data will come from APIs with free-tier request caps, which limits how often we can refresh real-time data per user session and pushes us toward caching results rather than querying live on every search.

**Security**  
Because itineraries tie a user's identity to specific dates and locations, publishing a trip in real time could reveal when someone's home is unoccupied, which is a genuine safety risk rather than a hypothetical one. To address this, itineraries default to private during the trip and only become visible to followers once the user marks the trip as completed, removing the real-time location signal while preserving the core feature of sharing detailed itineraries. This also reduces exposure of cost-breakdown data tied to an active, ongoing trip.

**Social**  
Voya's core social benefit is consolidating trip planning (currently scattered across weather sites, spreadsheets, and social media) into peer-sourced itineraries with transparent real costs, a genuine quality-of-life improvement for budget-conscious travelers rather than a novelty feature. Following travelers whose trips match your budget and style is meant to substitute for scrolling curated influencer content that rarely discloses real costs.

**Diversity and Cultural**  
We are targeting college-age and young-professional travelers with limited budgets, a group underserved by existing travel platforms that assume disposable income. Because itinerary content is user-generated, popular or wealthier destinations risk crowding out itineraries from lower-cost or less-visited regions, so we plan to surface itineraries by budget range and season rather than by popularity alone. This keeps the platform useful across a wider range of travel budgets and cultural contexts instead of converging on the same handful of trending destinations.

The clearest trade-off in our design is between security and the social feature that makes Voya useful in the first place: real-time sharing would make the platform feel more alive and social, but it directly increases physical safety risk, so we chose to delay visibility over maximizing real-time engagement.
