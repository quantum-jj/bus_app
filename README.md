# Bus Checker — v2.5.0

A lightweight London bus arrival checker PWA. Save your favourite locations, pick an origin and destination, and get live bus arrivals from nearby stops — no saved routes, everything is live. Designed as a quick daily commute companion, not a replacement for Citymapper or Google Maps.

Hosted at: `https://quantum-jj.github.io/bus_app`

---

## Features

### Location-based live lookup
Save favourite locations (home, work, gym, etc.) with addresses searched via Photon/OpenStreetMap. On the home screen, pick an origin and destination, then tap the search button to see live bus arrivals from nearby stops heading toward your destination.

### Smart search
Address and postcode search powered by Photon (OpenStreetMap) with automatic London-area filtering. UK postcodes are detected and resolved via postcodes.io for reliable results. Search is available everywhere: the location picker modal, the location editor, and settings.

### Smart route matching
The app finds bus stops near both your origin and destination, then cross-references which bus lines serve both ends. Only routes that actually connect the two locations are shown — no more irrelevant buses heading in vaguely the right direction. Results are grouped by stop, showing bus lines with up to 3 arrival times each, colour-coded by urgency.

### Configurable walking distance
Set your preferred walking radius in Settings (300m to 1.5 km). This controls how far from your origin and destination the app searches for bus stops — a larger radius finds more routes but means longer walks to/from stops.

### Walk times & maps
Each stop card shows an estimated walking time from your origin (haversine distance × 1.3 detour factor ÷ 80 m/min), plus a link to Google Maps walking directions.

### Auto-refresh
Results refresh automatically every 30 seconds while on the home screen, with a manual refresh button available too.

### Weather widget
Current weather conditions and air quality for your Home location (or a fallback area), powered by Open-Meteo. Includes weather alerts for extreme conditions. Tap to open BBC Weather.

### Custom emoji icons
Each saved location can have a custom emoji icon, displayed in the location picker, route picker, and settings.

### Location picker
A bottom-sheet modal with three ways to select a location: search for an address, use your current GPS position, or pick from your saved locations. Edit buttons let you jump straight to a location's settings.

### PWA with auto-updates
Installable as a PWA on iOS and Android. The app checks for updates on launch and every 4 hours, showing a toast notification when a new version is available.

---

## How to use

1. Open `index.html` in a browser (or install as a PWA via "Add to Home Screen")
2. Save your favourite locations in Settings (at minimum, set up Home and Work)
3. On the home screen, tap the origin selector and pick where you're travelling from
4. Tap the destination selector and pick where you're heading
5. Tap the round search button — live bus arrivals appear grouped by nearby stop

---

## Tech

- Plain HTML, CSS, and JavaScript — single `index.html` file, no frameworks, no build step
- **TfL Unified API** for live arrivals and nearby stop discovery
- **Photon** (OpenStreetMap) for address/location geocoding with London bias
- **postcodes.io** for UK postcode resolution
- **Open-Meteo** for weather and air quality data
- Dual-endpoint line matching (cross-references lines at origin and destination stops)
- Haversine formula + bearing calculations for distance, walk time, and direction sanity checks
- PWA manifest for home screen installation
- Dark iOS-style UI with CSS custom properties
