# RunRoute App Creation Guide

## Purpose

RunRoute is a single-file Progressive Web App that uses Google Maps to generate walking or running routes from a user-entered start point and target distance.

Main files:

- `index.html`: UI, Google Maps integration, routing logic, and route presentation
- `manifest.json`: PWA install metadata
- `sw.js`: service worker for offline caching
- `icons/`: home-screen icons

## Setup Instructions

1. Create a Google Cloud project.
2. Enable these APIs:
   - Maps JavaScript API
   - Directions API
   - Places API
   - Geocoding API
3. Create an API key.
4. Open the app and paste the key into the API key field.
5. Click `Apply Key & Load Map`.
6. Enter:
   - Pincode or area
   - Start point
   - End point, or enable `Loop route`
   - Distance
7. Click `Generate Today's Route`.

## Current Routing Model

The app now uses a midpoint-retrace flow for loop runs:

1. Read the user's total target distance.
2. Compute the one-way target as half of that distance.
3. Search for candidate midpoint or "vantage" locations around the start point.
4. Ask Google Maps for a walking route from the start point to each candidate.
5. Keep a candidate only when the one-way route is close to the half-distance target.
6. Present the total run as:
   - outbound distance to the midpoint
   - retrace the same path back
   - total distance = outbound x 2

For non-loop routes, the app still supports point-to-point routing.

## Common Errors And Remedies

### 1. `Please enter and apply your Google Maps API key first.`

Cause:

- The map script has not been loaded yet.

Remedy:

- Paste a valid key starting with `AIza`
- Click `Apply Key & Load Map`

### 2. `Please enter a valid Google Maps API key (starts with AIza)`

Cause:

- The key field is empty or malformed.

Remedy:

- Copy the full API key from Google Cloud Console
- Make sure there are no extra spaces before or after the key

### 3. `Could not find "<address>". Try being more specific.`

Cause:

- Geocoding could not resolve the start point or end point.

Remedy:

- Add the pincode or city name
- Use a more specific landmark or address
- Avoid abbreviations that are too local or ambiguous

### 4. Map loads, but route generation fails

Possible causes:

- Directions API is not enabled
- Billing is not enabled in Google Cloud
- The chosen area does not have a valid walking route

Remedy:

- Enable Directions API
- Confirm billing is active
- Try a nearby landmark on a public road or walkway

### 5. No midpoint route found near the requested distance

Cause:

- There may be no reachable walking route near the requested half-distance target from the chosen start point.

Remedy:

- Increase or decrease the requested distance slightly
- Try a different starting point
- Use a denser urban area or park entry instead of a vague locality

### 6. Loop route does not exactly match the target distance

Cause:

- Real roads and walking paths rarely line up with exact geometric distances.

Remedy:

- The app checks candidates within a tolerance window
- Try another variant or adjust the target by 0.5 to 1.0 km

### 7. `Open in Google Maps` behaves differently from the in-app route

Cause:

- Google Maps may re-optimize directions when opened externally.

Remedy:

- Use the in-app map as the primary preview
- Treat Google Maps as a navigation handoff, not a guaranteed pixel-perfect replay

### 8. Service worker or PWA install issues

Cause:

- The app is not being served over HTTPS, or the manifest/service worker paths are wrong.

Remedy:

- Use GitHub Pages or another HTTPS host
- Keep `manifest.json`, `sw.js`, and icon paths at the repo root as expected

## Development Notes

- The app is intentionally lightweight and keeps the main logic inside `index.html`.
- Places API can be used to search for candidate midpoints around the half-distance ring.
- Directions API is the final source of truth for actual route length.
- For retrace routes, the UI should clearly state that the return leg follows the same path back to the start.

## Suggested Future Improvements

- Persist API key locally with explicit user consent
- Add named route modes: `Retrace`, `Loop`, `Point-to-point`
- Show midpoint marker details on the map
- Let users choose stricter or looser distance tolerance
- Cache successful route candidates for faster retries
