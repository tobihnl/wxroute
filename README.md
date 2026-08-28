# wxroute

A weather briefing for a motorcycle route, not for a place.

Point forecasts answer "what is the weather in xyz". That is the wrong question when you are motorcycling, because you will be in one place at 13:40 and somewhere over a 1883 m pass at 11:10. This tool forecasts **every stop at the hour you actually arrive there**, compares four numerical weather models at each one, and shows you where they disagree.

Single HTML file. No build step, no dependencies, no backend, no API key. Everything runs in the browser.

## What it does

- Load one or more GPX files, or type a handful of coordinates
- Every stop shows its distance along the route and its elevation, which are always true
- Stops are named by reverse geocoding where a settlement is genuinely close, hedged to "near X" between 2.5 and 12 km, and left unnamed beyond that
- Tap any stop name to rename it to whatever you actually call it
- An optional map per route, with each stop coloured by its verdict
- Load several variants of the same trip and compare them side by side, cleanest one marked
- Drag the departure time from 05:00 to 16:00 and watch every arrival time and forecast re-time instantly
- A departure window bar scans the whole morning and marks the best start
- Per-route average speed, because a pass road and a motorway are not the same trip
- Routes persist in the browser, so a bookmark keeps your set

## Models

Served through [Open-Meteo](https://open-meteo.com/), which is a delivery layer over national meteorological services rather than a forecaster of its own.

| Model | Source | Resolution | Range |
|---|---|---|---|
| ICON-D2 | DWD | 2 km | ~48 h |
| ICON-EU | DWD | 7 km | 5 d |
| ECMWF IFS | ECMWF | 25 km | 10 d |
| ARPEGE | Météo-France | 11 km | 4 d |

Models outside their range are struck through automatically, with the reason shown. The high-resolution ones only appear as the ride gets close, which is the correct behaviour rather than a fault.

## How a stop is scored

| Condition | Level |
|---|---|
| 0.6–1.5 mm/h | Marginal |
| ≥ 1.5 mm/h | Poor |
| Gusts 40–60 km/h | Marginal |
| Gusts ≥ 60 km/h | Poor |
| CAPE 800–1500 after 11:00 | Poor |
| CAPE ≥ 1500 after 11:00 | Storm risk |
| 3–8 °C | Marginal |
| ≤ 3 °C | Poor |

Two rules matter more than the numbers:

**Consensus, not worst case.** A stop takes the upper median of the four model verdicts. One model saying 1.2 mm while three say zero does not turn the stop red. Two models agreeing does. The four squares under each stop still show every model individually, so the outlier stays visible instead of being averaged away.

**Median precipitation, with the spread.** The mm figure is the median across models. When the wettest model diverges by more than 0.4 mm it is appended after a slash, so `0.2 mm/1.4` tells you three models are dry and one is not. That disagreement is information.

CAPE is only counted after 11:00, since alpine convection is a diurnal signal. It says the atmosphere is willing, not that a cell will hit your road.

## Limitations

Worth knowing before you trust it:

- **No radar and no nowcast.** In the last few hours before departure, radar beats any model. Use this to plan, use radar to decide.
- **Place names are approximate by nature.** In remote terrain the nearest settlement can be kilometres away, so the map and the km marker are the reliable way to locate a stop. The name is a convenience, not a position.
- **Routes are sampled to nine stops** regardless of length. Fine for a 250 km day, coarse for a 700 km one.
- **Gridbox smoothing.** Even a 2 km model cannot resolve which side of a ridge a shower lands on. Precipitation in complex terrain is a probability spread over a valley, not a promise about your road.
- **Travel times are a speed estimate**, not routed. Straight-line distance is inflated by a winding factor of 1.30 for mountain roads and 1.16 for motorways. Adjust the per-route speed until the arrival times look right.
- **Gusts use the maximum across models**, not the median, deliberately. Wind outliers seemed worth being pessimistic about in a way rain outliers are not.

## Running it

Any static host works. It must be served over http(s) rather than opened as a `file://` path, because browsers block API requests from local files.

**GitHub Pages:** push this repo, then Settings → Pages → Source: deploy from branch, root. Your URL appears within a minute.

**Anything else:** copy `index.html` to any web space, or drop it on [Netlify Drop](https://app.netlify.com/drop) for a throwaway URL.

On iOS, Share → Add to Home Screen gives it an app icon and full-screen chrome.

## Data

Forecast data by [Open-Meteo](https://open-meteo.com/), licensed [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/), serving DWD ICON, ECMWF IFS and Météo-France ARPEGE.

Place names from [Nominatim](https://nominatim.openstreetmap.org/), map tiles from [OpenStreetMap](https://www.openstreetmap.org/copyright), both © OpenStreetMap contributors under the [ODbL](https://opendatacommons.org/licenses/odbl/). Lookups are throttled to one per second and cached permanently in the browser, so a route costs nine requests once and nothing thereafter. If you fork this and expect real traffic, move to a paid geocoder rather than leaning on the volunteer-funded service.

[Leaflet](https://leafletjs.com/) is loaded from a CDN and is the only dependency.

## Licence

MIT. See [LICENSE](LICENSE).
