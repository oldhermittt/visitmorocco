# Morocco Trip Designer *(working title)*

A web app that designs a complete trip to Morocco around each traveler's dates, budget, travel style and interests: a day-by-day circuit on an interactive map, with lodging, food and activities that fit the budget and travel times that are actually realistic.

Under the planner sits a curated directory of Moroccan hotels, riads, restaurants, activities and historic sites. This directory is also the foundation of the B2B offer.

---

## Why this exists

Generic trip planners and AI chatbots handle Morocco poorly:

- They underestimate travel times (Marrakech → Merzouga takes most of a day) and ignore that medinas are car-free.
- They don't account for Ramadan, seasonal extremes (desert summer, Atlas snow) or Friday rhythms.
- They miss practical cultural knowledge, such as which religious sites non-Muslims can visit.
- They don't help with the traveler's biggest anxieties: fair prices, scams, and finding a riad hidden in a medina.

This app is built specifically for Morocco, and its edge is depth and trust rather than breadth.

---

## Core experience

1. **Trip brief.** The traveler enters dates, total budget, group size, pace (relaxed / balanced / packed), and interests (desert, food, architecture, coast, crafts, history…).
2. **Generated circuit.** The app returns a day-by-day route on the map with:
   - lodging at each stop matched to budget,
   - sightseeing ordered by location and opening hours,
   - transport between stops (train, bus, grand taxi, private driver) with realistic durations,
   - a running cost estimate in MAD and the traveler's currency.
3. **Edit on the map.** Drag stops, swap a riad, add a day, make a day lighter. Route, timings and budget update live.
4. **Take it along.** Share, export to PDF / Google Maps, and use offline medina maps on the ground.

---

## Feature set

### Map and directory
- Morocco map with regions, provinces and cities
- Layers: hotels and riads, restaurants, activities, monuments and historic sites
- Filters by price tier, category, rating, distance
- Monument pages with short editorial stories and history (audio later)

### Trip planner
- Budget-based lodging suggestions
- Automatic circuit with multi-modal routing
- Overload warnings (too much driving, too many sites in one day)
- Season- and calendar-aware planning (Ramadan hours, heat, snow on mountain passes)
- Themed circuits: Imperial Cities, Northern Coast, Atlas & Desert, Jewish Heritage, Crafts, Food, Sufi music, World Cup 2030 match cities

### Trust and on-the-ground help
- Fair-price guide: taxi fares, haggling ranges, common scams
- Cultural notes per site (dress, access rules, etiquette)
- Offline medina maps and "meet point" pins for hard-to-find riads
- Clear "Sponsored" labeling for any paid placement

### Languages
English, French, Spanish, German, Arabic.

---

## B2B side

- **Free listings** for businesses at launch, to build supply
- **Paid tiers:** featured placement (labeled), listing analytics, direct booking or lead forwarding
- **Target customers:** small riads, independent licensed guides, local tour operators, restaurants
- **Later:** white-label circuit builder for travel agencies

Major businesses and all major monuments are listed by default. Businesses can claim and enrich their listing.

---

## Data strategy

| Data | Source | Notes |
|---|---|---|
| Monuments and historic sites | Wikidata + OpenStreetMap import | Curated and enriched with editorial content |
| Hotels (live prices) | Affiliate / booking API (Booking.com, Expedia, Hotelbeds-type) | Gives real budget matching plus affiliate revenue |
| Hotels (MVP fallback) | Manual price tiers (€ / €€ / €€€) | Until an API is integrated |
| Activities | GetYourGuide / Viator affiliate feeds + local B2B listings | |
| Restaurants | Manual curation | A short, verified list beats a large, stale one |
| Roads and travel times | OpenStreetMap Morocco extract | Used by the routing engine |
| Trains and buses | Manually maintained ONCF / CTM / Supratours data | Key intercity routes first |

Principle: **import what's public, curate what matters, verify what's shown.**

---

## Architecture

```
┌──────────────────────────────┐
│ Frontend (Next.js)           │
│ MapLibre / Mapbox GL         │
└──────────────┬───────────────┘
               │
┌──────────────▼───────────────┐
│ API layer                    │
│ - Directory / search         │
│ - Trip planner               │
│ - B2B dashboard              │
└──┬───────────┬───────────┬───┘
   │           │           │
┌──▼─────┐ ┌───▼──────┐ ┌──▼──────────────┐
│Postgres│ │ Routing  │ │ Itinerary engine│
│+PostGIS│ │ OSRM /   │ │ rules + LLM     │
│        │ │GraphHopp.│ │ (narration only)│
└────────┘ └──────────┘ └─────────────────┘
               │
   External: hotel / activity affiliate APIs
```

- **Postgres + PostGIS** (e.g. via Supabase) handles all geographic queries, for example "riads under €80 within 500 m of Jemaa el-Fna".
- **OSRM or GraphHopper** is self-hosted on the OSM Morocco extract for travel times and route geometry.
- **Itinerary engine:**
  1. Pick base cities from duration, interests and season
  2. Allocate nights per city
  3. Cluster sights per day by proximity and opening hours
  4. Select lodging per stop within the budget envelope
  5. Compute transport legs and validate daily load
  6. Use an LLM to write the narrative and handle edits like "make day 3 lighter". **The LLM may only choose places from the database**, which rules out invented locations.

---

## Data model (first pass)

- `region`, `province`, `city`: administrative hierarchy plus geometry
- `place`: shared base for all mappable items (name, i18n fields, location, category, city_id, status, source, verified_at)
  - `lodging`: type (hotel / riad / guesthouse / camp), price_tier, amenities
  - `restaurant`: cuisine, price_tier, hours, Ramadan hours
  - `activity`: duration, price, operator_id
  - `monument`: era, access rules, entry fee, story content
- `business`: B2B account, claimed places, plan tier
- `transport_link`: between cities, mode, duration, typical cost
- `trip`: user brief (dates, budget, pace, interests)
- `trip_day`, `trip_stop`: generated, editable itinerary

---

## Monetization (in order)

1. Affiliate commissions on hotels and activities
2. Paid B2B listing tiers and lead fees
3. White-label planner for agencies
4. Premium traveler features (offline packs, audio guides): optional

---

## Roadmap

### Phase 0: Validation
- Pick one launch region: **the North** (Tangier, Tetouan, Chefchaouen, Asilah) or **the classic loop** (Marrakech, Atlas, Merzouga)
- Hand-curate around 150–300 places and verify each one on the ground
- Test generated circuits with real travelers

### Phase 1: MVP
- Map + directory for the launch region
- Trip brief, then generated circuit, then map editing
- Price-tier lodging matching
- Monument pages with editorial stories
- EN + FR

### Phase 2: Revenue
- Hotel and activity affiliate integration with live prices
- Business claim flow and B2B dashboard
- Fair-price guide, offline medina maps

### Phase 3: Scale
- Add regions city by city
- Additional languages
- World Cup 2030 mode
- Agency white-label

---

## Open questions

- Launch region: North vs. classic loop?
- Product name and brand direction
- Which hotel API to start with (commission rates, approval requirements, caching rules)
- Accounts required to save trips, or anonymous planning first?
- How listings are verified and kept current (re-verification cadence, business self-updates)
- Policy on sponsored placement inside generated itineraries
