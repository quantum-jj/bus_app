# 🚲 Bike Checker

A minimalist iOS-style web app for checking Santander Cycles bike availability in London. Built as a single self-contained HTML file — no build tools, no frameworks, no dependencies.

**[Live App](https://quantum-jj.github.io/bikeapp)** · v1.1.0

## Features

- **Weather Widget**: Real-time weather with temperature, feels-like, wind, humidity, and AQI from Open-Meteo (no API key needed)
- **Severe Weather Warnings**: Automatically shows alerts for extreme cold, heat, high winds, or heavy rain/thunderstorms
- **Location Checker**: Quick view of bike/dock availability at your saved locations (Home, Office, Gym)
- **Route Planner**: Interactive From/To picker to check availability at both pickup and dropoff stations
- **Smart Backup Stations**: Automatically shows nearby alternatives only when main stations are low or empty
- **Customisable Stations**: Add, remove, and reorder primary and backup stations for each location from the settings menu
- **Customisable Weather Location**: Choose from 20 London areas for your weather data
- **iOS-Style UI**: Native dark mode app feel with `-apple-system` font, backdrop blur nav bar, and grouped list cells
- **PWA Support**: Add to Home Screen on iOS for a full-screen standalone app experience

## How It Works

The app makes a single API call to fetch all ~800 Santander Cycles stations, then filters locally for your saved stations. Data is cached for 30 seconds to avoid rate limiting.

Availability is colour-coded:
- 🟢 **Green**: Good availability
- 🟡 **Yellow**: Low availability (≤3 bikes when leaving, ≤5 docks when arriving)
- 🔴 **Red**: Empty/unavailable

Backup stations only appear when your primary stations are low or empty.

## Quick Start

### Hosted (recommended)
1. Visit the [live app](https://quantum-jj.github.io/bikeapp)
2. On iOS: tap Share → Add to Home Screen
3. Launch from your home screen like a native app

### Self-hosted
1. Fork/clone this repo
2. Enable GitHub Pages (Settings → Pages → Source: main branch)
3. Open your `username.github.io/bikeapp` URL
4. Add to Home Screen

## Customisation

All customisation is done through the in-app Settings menu (⚙️):

- **Weather Location**: Choose from 20 London areas
- **Stations**: Add/remove/reorder stations in each location group (Home, Office, Gym) by searching from the full TfL station list
- **Show/Hide Gym**: Toggle the Gym location on or off
- **TfL API Key**: Optional — raises rate limit from 50 to 500 requests/min. Get a free key from [api-portal.tfl.gov.uk](https://api-portal.tfl.gov.uk/)
- **Reset**: Clear all settings and return to defaults

### Station Limits

| Type | Min | Max |
|------|-----|-----|
| Primary stations | 1 | 4 |
| Backup stations | 0 | 4 |

## APIs Used

| API | Purpose | Key Required |
|-----|---------|:---:|
| [TfL BikePoint](https://api.tfl.gov.uk/) | Bike/dock availability | Optional (recommended) |
| [Open-Meteo Weather](https://open-meteo.com/) | Temperature, wind, conditions | No |
| [Open-Meteo Air Quality](https://open-meteo.com/) | European AQI | No |

## Tech Stack

- Vanilla JavaScript (zero dependencies)
- Single `index.html` file (~750 lines)
- CSS Variables for theming
- localStorage for settings persistence
- Progressive Web App manifest (inline base64)

## Browser Support

Optimised for:
- Safari (iOS/macOS) — primary target
- Chrome (Android/Desktop)
- Edge
- Other modern browsers with CSS Grid and Custom Properties support

## Known Quirks

- **Safari standalone mode**: Safari's Intelligent Tracking Prevention (ITP) can sometimes block cross-origin API calls in PWA mode. If the app shows "Load failed", try clearing Safari website data for the GitHub Pages domain and `api.tfl.gov.uk`
- **Rate limiting**: Without an API key, TfL limits you to ~50 requests per minute. The app batches all stations into a single request and caches for 30s, but rapid repeated taps can still hit the limit

## License

This is a personal hobby project. Feel free to use the code however you like.

## Contributing

Suggestions and improvements welcome — feel free to open an issue or PR!

---

**Note**: This app is not affiliated with Transport for London or Santander Cycles. It uses publicly available API data.
