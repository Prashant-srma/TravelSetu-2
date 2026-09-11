# TravelSetu — UI Refresh (September 2026)

Updated source code with the reference-inspired green sidebar layout, simpler pages, expandable day details, and complete Gemini-generated itineraries. Previous budget, journal, reviews, nearby services, Supabase and offline features remain. Start with `START-HERE.txt`; the optional native architecture is documented in `docs/ARCHITECTURE.md`.

## This update

Built on your latest `travel(1).zip`. See `UI-REFRESH.md` for the changes and verification. Keep your existing `.env.local` when updating.

## Start locally

Use Node.js 22.18+ or Node 24. Open Terminal inside the extracted folder **containing `package.json`**. Windows PowerShell users can use `npm.cmd` and `Copy-Item .env.example .env.local`; see `START-HERE.txt`. This ZIP has no extra nested project folder:

```bash
npm ci
cp .env.example .env.local
npm run dev
```

Open http://localhost:3000. This is the complete source project. `node_modules`, generated builds, personal records and secret keys are excluded. Both teammates install dependencies independently. Keep `.env.local` out of Git.

A Gemini API key is required to generate a new itinerary. Without credentials, discovery, existing saved trips, local journals, charts, downloads and sample marketplace still work. Internet is needed for public weather, nearby listings, street tiles and road routing. Accounts, cloud records, photo storage, public reviews and share links become available after Supabase setup. Gemini and traffic need their respective keys.

## Your requested stack

| Area | Implementation |
| --- | --- |
| Application | Next.js App Router, React, TypeScript / TSX |
| Design | Tailwind CSS, shadcn-style owned UI components using Radix Slot and CVA, Lucide React |
| Database | PostgreSQL on Supabase; trips, journals and expenses in JSONB documents; reviews and share links in relational tables |
| Authentication | Supabase Auth email/password signup, email confirmation and sign-in |
| Security | Row Level Security on all application tables; private storage policies; restricted public RPCs |
| Geographic data | PostGIS geography columns, spatial index and authenticated nearby-review function |
| Photos | Private Supabase Storage bucket; ten-minute signed viewing URLs |
| AI | Full Gemini-generated itinerary, streamed day previews, Zod validation; no silent preset fallback |
| Weather | Open-Meteo date-matched forecast |
| Map | Existing Google Maps interactive map and in-app navigation preserved; OpenStreetMap community POIs and offline overview retained |
| Road routing | OSRM route geometry, distance and driving-time estimates |
| Traffic and disasters | Optional TomTom incidents/routes; GDACS regional event screening |
| Analytics | Recharts planned-versus-recorded categories and cumulative spending |
| Export | jsPDF itinerary, standalone offline HTML, SVG overview map, CSV expenses, JSON journal, social PNG |
| Forms and state | React Hook Form, Zod, Zustand |
| API communication | fetch and Next.js route handlers |
| Deployment and collaboration | Vercel configuration, Git/GitHub CI workflow, npm lockfile |

## What is new

### Before your trip

- Reference-inspired green sidebar, compact header, instant destination search and a quieter dashboard. Explore retains 37 Indian destinations and catalog guides; Gemini can choose sights beyond those catalog entries.
- Start with **Local life**, **Balanced**, or **Classic highlights**. Local mode prioritises catalog neighbourhoods, regional food ideas and community experiences; it never assumes a business is locally owned without evidence.
- Preferences include arrival/departure times, daily start/end, budget scope, reserve, group composition, rooms, food/dietary notes, walking limits, accessibility, must-see places, exclusions, nearby radius and automatic adjustments.
- New trip dates default to **today in India (Asia/Kolkata)**. Old draft dates shift forward while preserving trip length; deliberately chosen future dates remain unchanged.
- Gemini creates every day and activity from your preferences: sights, meals, rest, local experiences, transfer blocks, timing, lodging estimates and detailed notes. The app validates dates, overlaps, departure, walking and costs. No activity JSON catalog is supplied as a preset schedule. A missing key, provider error or invalid result shows a retry/setup error; no demo plan is substituted.
- The itinerary opens on a clean day timeline. Expand an activity for full details; use separate Food & Stays, Conditions, Budget and Downloads tabs. Conditions continue refreshing in the background while the itinerary is open.
- Model-proposed coordinates are labelled Gemini estimates; distant/unusable coordinates are omitted. Unmapped sightseeing places can still be reviewed. Map pins are not verified entrances.

### During your trip

- Conditions panel shows provider, checked time, coverage, actual reports and a change log. It refreshes every five minutes while visible and supports manual refresh, review-only suggestions and applying changes.
- Severe forecast triggers pause exposed activities on the matching date. Valid current regional events and nearby road reports can prompt conservative changes. Traffic-aware route durations can extend transfer blocks; visits that no longer fit are deferred while departure remains protected.
- Missing, stale or out-of-window reports remain **unavailable / unassessed**. No synthetic weather or disaster information drives the itinerary. Previously paused activities are not silently restored when the relevant feed cannot be rechecked.
- These adjustments do not cancel tickets, reservations or payments. Elapsed activities are preserved. Current GDACS events are not treated as predictions of future disasters. GDACS is broad regional screening, not an exhaustive India warning service; check official local instructions and https://sachet.ndma.gov.in/.
- Nearby hotels, restaurants, experiences and transport use named OpenStreetMap listings with coordinates, ranked by distance to planned stops and local/dietary tags. Distances are straight-line, not walking directions. Refresh after changing the plan. Unknown ratings, ownership and hours are not invented.
- Map filters include category, day, radius, text, access, hours and other available attributes. **Load & save driving route** retrieves OSRM geometry and duration. Solid routes are saved road estimates; dashed lines show stop order. OSRM has no live traffic or live closure knowledge. Generated transfer times are model estimates unless the optional traffic provider supplies a usable duration.

### Budget and post-trip

- **Budget:** create/edit/delete expenses by date, category and payer; integer-paise calculations; category comparisons; cumulative spending; group and per-person totals; reserve and over-budget display; downloadable CSV.
- Recorded spending is not inferred from bookings. Zero means nothing has been recorded. Planned costs are estimates. The reserve is displayed separately from expenses.
- **Post-Trip:** visited-place checklist, completion status, private reflection, hotel/restaurant/sight reviews, private photo album, recap and full journal backup. Original planned sights remain available for review even after replanning.
- Reviews support 1–5 stars, visit date, text, edit/delete and explicit public/private choice. Star clicks save to the device; **Save review to account** persists and optionally publishes. Public averages cover up to 50 recent loaded reviews and are traveller opinions, not verified bookings. No author identity or email is exposed by the public review RPC.
- Social sharing exports a 1200px PNG, caption and native share sheet when supported. Account users can create a 30-day link and open WhatsApp/LinkedIn sharing. Spending is opt-in; dates, private notes, names, photos, contacts and itemised expenses are excluded. Links can be revoked individually or for the entire trip; downloaded copies cannot be recalled.
- Account-linked journal changes retry syncing every minute while online and on reconnect. First upload is explicit. Optimistic database versions prevent silent overwrites across devices; edits made during a save stay pending. If a conflict occurs, download the private journal and use **Keep as a separate trip copy** to preserve both versions.

### Offline resilience

1. Generate a trip and refresh its nearby listings while online.
2. Open **Trip Map**, select a day, and load/save its driving route if available.
3. In the itinerary or journal, download the **offline pack**. Optionally download the PDF, SVG map and emergency contact text.
4. Open the downloaded HTML without internet. It contains the itinerary, saved stop map, cached route geometry when available, nearby listings and emergency contacts.

This is a standalone offline overview, not downloadable street-map tiles or offline turn-by-turn navigation. No bulk downloading of OpenStreetMap tiles is performed. The service worker supplies a basic saved-plan fallback if the site is reopened without a connection. Already-loaded journal pages can keep local edits. Live alerts, accounts, booking searches and new route requests need internet; telephone calls need a working telephone connection. Exported files are snapshots and do not update themselves.

## Supabase setup

1. Create a Supabase project. In the SQL editor run `supabase/migrations/001_travelsetu.sql` **once**, on a fresh database. It creates tables, PostGIS, RLS, RPCs and the private `trip-photos` bucket. If your project already has PostGIS in another schema, adapt the `extensions` qualification before applying.
2. Set the project URL and public anon key in `.env.local`:

```dotenv
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_public_anon_key
```

3. Enable email/password authentication. Configure the Site URL and allowed redirect URLs in Supabase Auth, including `http://localhost:3000/account` and your deployed `/account` URL. Configure your email sender for production confirmation delivery.
4. Restart/rebuild Next.js after changing public environment variables. Sign up, follow the email confirmation and sign in.
5. Generate a trip. Open Budget, Post-Trip or Account and choose **Save to account**. Reviews have their own explicit save button. Photos are private by default and limited to JPG/PNG/WebP, 5 MB each, 20 per trip in the app.
6. In a development project, run `supabase/tests/rls.sql` to check cross-user read/write isolation, public/private reviews, PostGIS queries, anonymous restrictions and expiring share links. The script rolls back its fixtures.

Never expose a service-role key in browser variables. No service-role key is required by this implementation. RLS protects cloud records; locally cached data and exported files are stored on the user's device. A signed-out account's journal is hidden in the app, but device backups can remain in browser storage.

## Provider configuration

### Gemini

```dotenv
GEMINI_API_KEY=your_google_ai_studio_key
GEMINI_MODEL=gemini-2.5-flash
```

Set a structured-output model available to your account; the default is `gemini-2.5-flash`. The itinerary endpoint uses that model and streams previews via SSE. Missing credentials, quota/model errors or invalid schedules require a retry or configuration fix. It does not substitute another itinerary. Gemini uses JSON as its response format so TypeScript can validate and render it; JSON files do not supply the generated schedule. Guidance is not confirmation of hours, permits, allergy safety or availability.

If you see a Gemini finish/access error, first set `GEMINI_MODEL=gemini-2.5-flash`, restart `npm run dev`, and try a short 1-2 day trip. Remove old `DEMO_MODE` lines from `.env.local`; new itinerary generation ignores them. If the error says the key was rejected, create a fresh Gemini API key in Google AI Studio and replace `GEMINI_API_KEY`.

### Conditions and maps

Open-Meteo, GDACS and the public Overpass/OSRM endpoints do not require keys in this setup. They can be unavailable or rate-limited and have no application SLA. Cache windows and query caps reduce load. For traffic, add `TOMTOM_API_KEY`. TomTom coverage is bounded around the trip, and routing covers the supported near-term departure window; later legs are estimates and should be refreshed during travel. For larger deployments, configure a dedicated HTTPS `OSRM_BASE_URL` and provider plans appropriate to expected usage. Weather risk thresholds are conservative application heuristics, not official warning classifications.

### Hotelbeds hotel and airport-transfer APIs

Set separate Hotel and Transfer evaluation credentials from https://developer.hotelbeds.com/documentation/getting-started/:

```dotenv
HOTELBEDS_HOTEL_API_KEY=
HOTELBEDS_HOTEL_SECRET=
HOTELBEDS_TRANSFER_API_KEY=
HOTELBEDS_TRANSFER_SECRET=
BOOKING_SIGNING_SECRET=your_long_random_secret
```

Use **Book Travel → Supplier test inventory**. Hotels can be searched around an itinerary stop or selected nearby hotel with a bounded radius. Occupancy is adults/children per room with identical occupancy across rooms. For airport transfers, select pickup airport, flight time and the actual supplier hotel drop-off. Search, review the refreshed quote and confirm using supplier-approved test guest details.

Implemented Hotelbeds endpoints: hotel availability, rate check, test booking, airport-transfer availability and test booking. All use `https://api.test.hotelbeds.com`. The app API is `/api/bookings` (`search`, `quote`, `confirm`). Signed expiring quotes retain supplier currency and terms. Empty supplier results remain empty. The expanded demo catalog has 8 hotel options, 9 vehicle types and 7 bus options, filtered by capacity and query.

**Booking scope:** no real reservations, payments, daily car rental API or bus-ticket API is enabled. Hotelbeds is evaluation-only; rentals, buses and flights are labelled samples. Local test confirmations use a filesystem journal to prevent duplicate attempts and preserve unknown outcomes. Vercel supports searches/quotes but deliberately disables confirmations because that journal is not durable across serverless instances. Production supplier booking requires certification, durable transactional storage, production operations and cancellation/payment workflows.

## Vercel and GitHub

The project includes `vercel.json` and `.github/workflows/ci.yml`. Push the source to your own GitHub repository and import that repository into Vercel with the Next.js preset. Set the environment variables in Vercel, configure Supabase redirect URLs for the deployed domain, then deploy. A public deployment is required for recipients to open share links; localhost links are local only.

The itinerary API declares a 180-second maximum duration to accommodate provider requests; your Vercel plan must support the configured duration. This deliverable does not create a GitHub repository or deploy to a Vercel account. Provider quotas, email delivery and cloud configuration remain account setup steps.

## Validation

```bash
npm test
npm run typecheck
npm run build
```

37 automated application tests cover schedule and budget integrity across all destinations, dates and time zones, local preference, weather date matching, provider outages, stale/partial conditions, idempotent replanning, nearby filters, signed booking quotes and duplicate confirmations, mocked supplier/OSRM requests, integer-paise expense totals, share whitelisting, offline HTML escaping and sync race protection.

The 37 application tests, six native/limiter/cache tests, both TypeScript checks, native C++ compilation and Next.js production build passed. The full gateway/IPC smoke test could not run because the environment blocks Unix sockets. See the architecture guide for unverified optional integrations. Historical `PHASE-*.md` files describe earlier versions; this README documents the current implementation.

No credentials were supplied for live Gemini, Supabase, TomTom or Hotelbeds account tests. Cloud migrations and the included SQL isolation checks must be run in your Supabase project; they were not executed against a live database here. Browser visual testing was not run in this environment.

## Key files

- `app/plan-trip`, `app/itinerary`, `app/replanning`: planning and adaptive itinerary UI
- `lib/conditions`, `lib/weather.ts`: provider status, weather and change logic
- `lib/nearby.ts`, `app/nearby`, `app/marketplace`: nearby services and test supplier integration
- `app/components/TripMap.tsx`, `lib/routing.ts`: MapLibre and OSRM
- `app/components/TripJournal.tsx`, `lib/journal.ts`: budgets, journals and exports
- `lib/travel-store.ts`, `lib/cloud.ts`: Zustand, persistence and versioned sync
- `app/components/PlaceRating.tsx`: reviews and community ratings
- `app/components/SharePanel.tsx`, `app/share/[token]`: social exports and share links
- `lib/offline.ts`, `public/sw.js`, `public/offline.html`: offline resources and PDF
- `supabase/migrations`, `supabase/tests`: database, security and verification

## Data and documentation

Catalog coordinates and entry costs are approximate editorial data. The discovery catalog has four named sights per destination; the Gemini generator can propose other sights independently. Local food/dietary suggestions require ingredient confirmation. Nearby community data may be incomplete. Map routes and weather reports do not establish that a route is safe.

Photo authors/licenses are in `data/image-credits.json` and the site's credits page. New Wikimedia images retain CC BY-SA 4.0 attribution; other destination images come from the supplied project. Map data © OpenStreetMap contributors, ODbL.

Primary integration references:

- https://ai.google.dev/gemini-api/docs/generate-content/structured-output
- https://supabase.com/docs/guides/database/postgres/row-level-security
- https://supabase.com/docs/guides/database/extensions/postgis
- https://supabase.com/docs/guides/storage/security/access-control
- https://open-meteo.com/en/docs
- https://docs.tomtom.com/routing-api/documentation/tomtom-maps/v1/calculate-route
- https://docs.tomtom.com/traffic-api/documentation/tomtom-maps/v1/traffic-incidents/incident-details
- https://www.gdacs.org/Documents/2025/GDACS_API_quickstart_v2.pdf
- https://wiki.openstreetmap.org/wiki/Overpass_API
- https://maplibre.org/maplibre-gl-js/docs/
- https://github.com/Project-OSRM/osrm-backend/blob/master/docs/http.md
- https://developer.hotelbeds.com/documentation/hotels/booking-api/workflow/
- https://developer.hotelbeds.com/documentation/transfers/booking-api/search-availability/availability-simple/
- https://vercel.com/docs/functions/configuring-functions/duration
- https://tourism.gov.in/ (Ministry of Tourism tourist helpline: 1363 / 1800 11 1363; national emergency: 112)

## Optional gateway and native compute

`services/gateway` contains a separately installable TypeScript/uWebSockets.js service, optional RESP cache, WebSocket search/weather subscriptions, Gemini SSE proxy and local IPC bridge. `services/compute` contains C++20 distance/order and SIMD dot-product kernels, optional CUDA source, a llama.cpp CLI adapter and an optional libpqxx report. These are a self-hosted foundation, not a benchmarked enterprise compute/HTAP system. The normal Next.js site works without these services. Full setup, protocol corrections and verification limits are in `docs/ARCHITECTURE.md`.

Apply `supabase/migrations/002_analytics.sql` for owner-scoped SQL expense totals after migration 001. The pgvector schema is opt-in in `supabase/optional-vector.sql`. Existing data is retained; do not rerun migration 001 on an already-migrated database.

## Trip tracking and budget choices update

This update extends the existing app and preserves its layout and integrations.

- **Trip map:** Start GPS tracking, pause/resume, follow the device, mark visited stops, fit the recorded route, and create a map PNG with distance, recording time, visit count and OpenStreetMap attribution. Preview before downloading or using the device share sheet. Browsers need HTTPS (or localhost) and location permission. GPS recording is foreground-only, stored on the device per trip, and does not promise background tracking while the phone is locked. Poor fixes and large gaps are excluded from distance calculations. Clear the recording from Recording details.
- **Near my location:** Looks up real OpenStreetMap hospitals and police within 5 km of a recent device fix, sorts by straight-line distance, and opens directions with the actual origin/destination coordinates. It refreshes after movement and periodically while GPS remains fresh. The search sends coordinates to the server and Overpass; precise device searches are not cached by the application. Incomplete or failed searches never create fictional emergency locations. Dial 112 for an emergency in India.
- **Itinerary dropdowns:** Each day's hotels, restaurants/cafés, local activities, local transport and first-day return-travel allowance have compact expandable choices. Real OSM listings include directions and reviews but have no verified prices. Unpriced listings are explicitly separated from affordable demo examples. Demo JSON prices in `data/recommendation-demo.json` are fictional, scale by travellers/rooms/vehicles, and fit the individual block allowance and total budget. They are not reservations or actual live fares. Existing Hotelbeds integrations remain test-only.
- **Budget guard:** Gemini chooses the itinerary content; the application independently sums group costs and the reserve. Over-budget generation is rejected rather than silently reducing prices. The planner offers a suggested revised budget for that proposed plan or cheaper preferences. Budget edits on the itinerary page preserve the saved plan and open a draft for regeneration. Actual provider prices still need confirmation and are not guaranteed by AI estimates.
- **Reviews:** Sightseeing, food, stays, transport and manually marked visits can be reviewed. Device reviews work without setup. Account syncing/public reviews use the existing Supabase schema and authentication; public ratings are traveller opinions, not verified-stay ratings.

Run `npm run check:gemini` after setting `.env.local` to check the configured key, model and structured-output support. It makes one small request to Google and prints a specific diagnostic without printing the key. Restart the server after changing environment variables. This update uses `responseJsonSchema` for the Zod-generated JSON schema. Model/access, authentication, quota, network, invalid output and budget failures have distinct handling. No sample itinerary replaces a failed Gemini response.

The shared source ZIP intentionally excludes `.env.local`, build outputs, installed dependencies, recordings, reviews and booking journals. Keep your own `.env.local` when updating your project. This update passed desktop/mobile browser checks, including vehicle budget upgrades and Explore filters. Weather visuals were tested with controlled fixtures. Live Gemini credentials, Google Maps, Supabase and supplier transactions were not verified; use the connection check on your machine.
