
# iNatGaps

A map tool for finding the places where nobody has been recording nature.

iNaturalist observations pile up where people already go: trailheads, city parks, the roads between them. iNatGaps divides an area into a grid, counts the observations in each square, and ranks the squares by how little they've been documented. If you're deciding where to spend some time looking for life, the empty squares are where your records will provide the most benefit to citizen science.

It's a single HTML file. There is no server, no build step, and nothing to install.

**Live version:** `https://als.github.io/iNatGaps/`

## What it does

1. Search for a place, upload a boundary, or draw a box on the map.
2. Pick a grid size, a taxon group, and/or a date range.
3. Run the scan. Each square is colored by how many observations it contains, from "zero records" to "most records".
4. Click a square (or pick one from the ranked list) to see its observation count, species list, and number of observers, then jump to iNaturalist to explore it.
5. Export the results as CSV or GeoJSON.

Counts only include observations iNaturalist marks as "verifiable" (with a date, a location, and a photo or sound).

## Choosing a region

- **iNaturalist places.** Start typing and pick a result. The grid is clipped to the place's outline when iNaturalist has one.
- **OpenStreetMap search.** If iNaturalist doesn't know the place, search OpenStreetMap from the bottom of the dropdown. Good for smaller parks, reserves, and neighborhoods.
- **Upload a boundary.** GeoJSON and KML files work. Anything with polygons is fine, including multi-part shapes and holes. Very detailed outlines are simplified automatically so the map stays responsive.
- **Draw a box.** Click "Draw boundary" and drag a rectangle.

## Scan modes

**Standard** queries iNaturalist once per grid cell. Grid sizes run from 100 m to 150 km, and a scan is capped at 150 cells. By default it uses a "skip empty areas" approach: it checks large blocks first and only drills into blocks that contain observations.

**Fast mode (beta)** counts whole map tiles instead, so one request fills up to 1,024 cells. That allows much larger areas and finer grids: up to 40 tiles and 8,000 cells per scan. Cells follow the web-map tile grid, so you choose a zoom level rather than a size in kilometers, and cells get slightly smaller toward the poles. The CSV and GeoJSON exports include each cell's area so you can compare density.

Fast mode has a small diagnostics panel that compares tile counts against iNaturalist's observation counts for the same area. In my own checks, a tile of about 4,600 observations differed by 0.1% and a tile of 1.9 million differed by 0.3%. Individual cells I spot-checked matched exactly. The small differences are probably down to observations added between requests, but I haven't confirmed that.

## Map layers

Esri street, topographic, and satellite basemaps, plus an optional place-label overlay for the satellite view. Satellite imagery is useful for working out why a square is empty: open water, a golf course, a fenced facility.

There is also an optional **Public lands (beta)** overlay for the United States. The first time you switch it on, the app tries a short list of public map services and uses the first one that responds. None of those addresses are guaranteed to stay up, so if it can't find one it will say so and turn itself off.

## How the priority score works

Priority is calculated from observation counts only. Each square is scored against the busiest square in the same scan, on a log scale, so a square with few records scores near 100 and the busiest square scores 0. It is a rough guide to where records are thin. It doesn't know about habitat, terrain, or whether you're allowed to be there.

A low count means few records, not low biodiversity. Always check land access and safety before you go.

## Where the data comes from

All requests go straight from your browser to public services. There is no backend and nothing is collected.

| Service | Used for |
| --- | --- |
| [iNaturalist API](https://api.inaturalist.org/v1/docs/) | Observation counts, species lists, place search and outlines |
| iNaturalist grid tiles | Fast mode counts |
| [OpenStreetMap Nominatim](https://nominatim.org/) | Optional place search and outlines |
| Esri | Basemaps, satellite imagery, place labels |
| BLM / USGS / Esri services | Optional public lands overlay |
| [Leaflet](https://leafletjs.com/) | The map |

No API keys are needed.

Requests to iNaturalist are paced to stay inside their rate limits, and the app backs off if it gets a 429 response. OpenStreetMap searches only run when you click, at most once a second, as Nominatim's usage policy asks.

## Privacy and storage

The app stores a few things in your browser's `localStorage`: recent observation counts and decoded tiles (kept for 24 hours so repeat scans are fast) and whether the summary cards are collapsed. Nothing is sent anywhere except the lookups above. The "Fresh" button ignores the saved counts and queries again.

Shared links store the place, grid size, taxon, and dates in the URL hash. For places from OpenStreetMap or an uploaded file, a shared link keeps only the bounding box, not the exact outline.

## Running it

Open `index.html` in a browser.

You need an internet connection, since the map, fonts, and data all load from external services.

## Known limitations

- Uploaded shapefiles (`.zip`, `.shp`) aren't supported yet. Convert them to GeoJSON first, for example at [mapshaper.org](https://mapshaper.org).
- Uploaded files need longitude/latitude coordinates (WGS84). Projected coordinates are rejected with a message.
- iNaturalist hides the exact location of observations of sensitive species, so a few records may sit in a neighboring square.
- The public lands overlay is US-only and depends on third-party services.
- Esri and OpenStreetMap have their own terms for map tiles and search. Check them if you plan to run this for a lot of people.

## Credits

Observation data belongs to iNaturalist and its contributors, and each observation keeps its own license. Place outlines from OpenStreetMap are © OpenStreetMap contributors.

## License

This project is licensed under the **GNU General Public License v3.0** (GNU GPLv3). See the [`LICENSE`](LICENSE) file for full details.
