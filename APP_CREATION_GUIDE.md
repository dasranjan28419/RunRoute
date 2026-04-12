# RunRoute App Creation Guide

## Purpose

RunRoute is a single-file Progressive Web App that uses open-source mapping services to generate walking or running routes from a user-entered start point and target distance.

Main files:

- `index.html`: UI, Leaflet/OpenStreetMap rendering, routing logic, and route presentation
- `manifest.json`: PWA install metadata
- `sw.js`: service worker for offline caching
- `icons/`: home-screen icons

## Setup Instructions

1. Open the app.
2. Let the built-in open-source map initialize, or tap `Reload Open Map` if the map needs a fresh start.
3. Enter:
   - Pincode or area
   - Start point
   - End point, or enable `Loop route`
   - Distance
   - If start and end are the same, choose `Looped Path` or `Retraced Path`
4. Click `Generate Today's Route`.

Each generated route is stored locally in the browser with its route type and total distance so the user can scroll through previously generated routes and reload a preselected one later.

On first use on a device, the app should also encourage the user to export a backup file to Files or Downloads. That backup can then be imported on the same device later or moved to a different device.

The app also includes an in-app install help card so iPhone users see the Safari `Add to Home Screen` path and Android users can use the browser install flow when available.

## Routing Algorithm

The routing logic should be handled in two separate cases.

### Case 1: Start point and end point are the same

When the user selects the same address for both start and end, the app can support two same-location route styles.

#### Case 1A: Closed loop selected

This mode should only be used when the start and end address are the same.

In the UI, this is exposed through a dedicated `Looped Path` button that appears only for same-address routes.

1. Read the user's total target distance.
2. Compute the per-loop target distance.
3. Build candidate rectangles using the start address as one corner.
4. Use a base long edge of about one-third of the per-loop target distance, increased by 20 percent to search a wider region.
5. Use a shorter perpendicular edge, also increased by 20 percent, so the rectangle can close into a realistic loop.
6. Trace the other three rectangle corners from the start corner.
7. For each traced corner, find the nearest real stop point by snapping to a nearby address, house, landmark, or road-accessible place.
8. For each snapped rectangle, request a route that:
   - starts at the given address
   - visits the three snapped rectangle corners only once
   - returns to the same start address
9. Measure the full closed-loop distance returned by the routing service.
10. Compare that distance against the requested target distance.
11. If multiple rectangle candidates are valid, rank them by distance match and keep the optimal routes first.
12. Present the best loop options to the user.

When enough nearby matches exist, the option list should be centered on the requested distance:
- two options below the requested value
- one center option closest to the requested value
- two options above the requested value

This is the preferred same-location mode when the goal is a true loop without retracing the same path back.

#### Case 1B: Retrace selected

This remains the fallback out-and-back model:

1. Read the user's total target distance.
2. Compute the one-way target as half of that distance.
3. Create a ring of candidate destinations around the start point using that half-distance radius.
4. Ask the routing service for traced walking routes to those ring candidates.
5. Treat every route turn as a possible turnaround or "vantage" point.
6. Add route length in increments using each turn step distance.
7. Convert each cumulative outbound distance into a total retrace distance:
   - total = outbound x 2
8. Keep the options that fall within `+/- 2 km` of the user's requested distance.
9. Present those options in the route selector.

When enough nearby matches exist, the option list should be centered on the requested distance:
- two options below the requested value
- one center option closest to the requested value
- two options above the requested value

This mode is appropriate when the user starts and finishes at the same location and is comfortable retracing the same path back to the origin.

In the UI, this is exposed through a dedicated `Retraced Path` button that appears only for same-address routes.

### Case 2: Start point and end point are different

When retrace is enabled but the user enters different start and end points, the app should not use the half-distance method. Instead, it should search for the optimal route that matches the requested total distance as closely as possible, with exact distance as the primary goal.

1. Read the user's start point, end point, total target distance, and route preferences.
2. Geocode both endpoints and build the direct walking route between them.
3. Measure the direct route distance from start to end.
4. Compare the direct route distance to the user's requested target distance.
5. If the direct route already matches the requested distance, use it as the result.
6. If the direct route is shorter than the target, compute the extra distance that must be added.
7. Generate candidate shaping routes between start and end by exploring:
   - corridor waypoints near the direct route
   - lateral detours on either side of the route
   - parks, blocks, and reachable road segments that can extend distance without breaking the final destination
8. Request routed paths for each candidate from start to end through those additional waypoints.
9. Score each candidate using distance error as the highest-priority metric:
   - primary score: absolute difference from the target distance
   - secondary score: fewer unnecessary detours or awkward backtracking
   - tertiary score: better route quality for walking or running
10. Prefer the candidate that produces the exact requested distance. If an exact match is not available on the road network, choose the closest feasible route within the tolerance window.
11. Present the best route options in ranked order so the user can choose the most suitable one.

In this second case, the route still ends at the user-selected destination, but the distance is achieved by optimizing the full path between the two endpoints instead of assuming a half-distance turnaround.

## Common Errors And Remedies

### 1. `The open-source map is still loading. Reload it and try again.`

Cause:

- The Leaflet or tile layer setup has not finished yet.

Remedy:

- Tap `Reload Open Map`
- Refresh the app if the map assets were blocked during first load

### 2. `Could not find "<address>". Try being more specific.`

Cause:

- Geocoding could not resolve the start point or end point.

Remedy:

- Add the pincode or city name
- Use a more specific landmark or address
- Avoid abbreviations that are too local or ambiguous

### 3. Map loads, but route generation fails

Possible causes:

- The public routing service is temporarily busy
- The public geocoding service rejected a burst of requests
- The chosen area does not have a valid walking route

Remedy:

- Wait a moment and try again
- Try a nearby landmark on a public road or walkway

### 4. No loop, retrace, or exact-distance option found near the requested distance

Cause:

- There may be no four-point loop, traced turn combination, or waypoint-based path that lands inside the target window from the chosen start and end points.

Remedy:

- Increase or decrease the requested distance slightly
- Try a different starting point
- Use a denser urban area or park entry instead of a vague locality

### 5. Loop route does not exactly match the target distance

Cause:

- Real roads and walking paths rarely line up with exact geometric distances.

Remedy:

- The app checks candidates within a tolerance window
- Try another variant or adjust the target by 0.5 to 1.0 km

### 6. `Open in Google Maps` behaves differently from the in-app route

Cause:

- Google Maps may re-optimize directions when opened externally.

Remedy:

- Use the in-app map as the primary preview
- Treat Google Maps as a navigation handoff, not a guaranteed pixel-perfect replay

### 7. Service worker or PWA install issues

Cause:

- The app is not being served over HTTPS, or the manifest/service worker paths are wrong.

Remedy:

- Use GitHub Pages or another HTTPS host
- Keep `manifest.json`, `sw.js`, and icon paths at the repo root as expected

## Development Notes

- The app is intentionally lightweight and keeps the main logic inside `index.html`.
- For same-location loop mode, the app generates enlarged rectangles from the start address, then snaps the other corners to nearby real stop points.
- Reverse geocoding is used to snap rectangle corners to nearby real stop points.
- For different start and end points, waypoint sampling is used to build candidate detours along the route corridor.
- OSRM route distance is the final source of truth for actual route length.
- For retrace routes, the UI should clearly state that the return leg follows the same path back to the start.
- For closed-loop routes, the UI should clearly state that the route visits the snapped rectangle corners once and returns to the start address.
- Generated routes are persisted in browser local storage so they can be reloaded from the saved-routes list.
- The app also supports exporting saved routes to a JSON backup file and importing that file later.
- On mobile devices, the actual save location is chosen by the user through the browser or OS share/save flow, not by the app automatically.
- The UI includes author/contact details for Ranjan Das at `das.ranjan28419@gmail.com`.
- The app now runs without a Google API key and can initialize its map immediately with open-source services.
- On mobile layouts, the live route map is repositioned directly below the generate button, with the `Open in Google Maps` action shown immediately beneath it.

## Suggested Future Improvements

- Add named route modes: `Retrace`, `Loop`, `Point-to-point`
- Show midpoint marker details on the map
- Let users choose stricter or looser distance tolerance
- Cache successful route candidates for faster retries
- Move public routing/geocoding behind a small proxy or self-hosted service for better reliability and rate-limit control
