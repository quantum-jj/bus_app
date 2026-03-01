# Bus Checker — v1.9.0

A minimalist London bus tracker PWA. See live arrival times for your favourite routes, organised into groups by location. Installable on iOS and Android for a native app feel. Dark, iOS-style UI — no dependencies, no build step.

Hosted at: `https://quantum-jj.github.io/bus_app`

---

## Features

### Route groups
Favourites are organised into **groups**, each anchored to an origin postcode (e.g. "Home", "Work"). Within each group you can track as many bus routes as you like. Groups and routes can be reordered via drag-and-drop.

### Live arrivals
Each tile shows the next 3 arrival times for that route and stop, pulled from the **TfL Unified API**. Arrivals marked with a dot are live-tracked; others are scheduled. The tile shows a timestamp of the last refresh and a manual refresh button.

### Night bus merging
If your route has a night bus equivalent (e.g. 205 ↔ N205), the app automatically fetches arrivals for both and merges them into a single tile. When buses from both variants are in the next-3 window, the route badge cycles between the two line names with a smooth animation.

### Walk time
Each tile estimates your walking time from the group's origin postcode to the bus stop using a haversine distance calculation (with a 1.3× detour factor at 80 m/min). This is displayed inline on the stop name line. Walk times are computed once and stored — legacy routes get a retroactive estimate if stop coordinates are available.

### Google Maps link
Every tile has a 📍 button that opens Google Maps walking directions from your group's origin postcode to the bus stop.

### Collapse / expand
Individual tiles can be collapsed (showing just the route badge, stop name, and towards destination) by tapping the chevron in the top-right corner. Entire groups can also be collapsed by tapping the chevron next to the group name.

### Location awareness
Enable location in the settings to automatically detect which group is nearest you. Nearby groups are shown first, sorted by your custom group order. Non-nearby groups follow, also in your custom order. On first location detection, non-nearby groups are auto-collapsed for a cleaner view. A green dot appears on nearby group headers.

### Group management
- **Add / edit groups** — create a group with a name and origin postcode via the Settings page
- **Drag to reorder** — drag groups up or down in Settings to set your preferred order
- **Add routes via wizard** — a two-step wizard searches TfL stops near your postcode for your chosen line and direction
- **No-results fallback** — if no routes are found (e.g. outside service hours), you can still save the group as a placeholder and add routes later
- **Cross-group routes** — a route that already exists in another group can still be added to the current group; routes in other groups are marked "in another group" in the wizard

---

## How to use

1. Open `index.html` in a browser (or install as a PWA via "Add to Home Screen")
2. Tap **＋ Add group** and enter a name and your origin postcode
3. The wizard searches nearby stops — select the route and direction you want
4. Your tile appears on the home screen; tap **↻** to refresh arrivals or pull to re-fetch all

---

## Tech

- Plain HTML, CSS, and JavaScript — single `index.html` file, no frameworks, no build step
- **TfL Unified API** for live arrivals and stop data
- **postcodes.io** for postcode geocoding
- **TfL Journey Planner API** for walk-time estimates during route setup
- Haversine formula for distance calculations (walk time, proximity detection)
- Pointer Events API for drag-and-drop (route and group reordering)
- PWA manifest + service worker for offline capability and home screen installation
