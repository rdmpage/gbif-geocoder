# CLAUDE.md

Project documentation for the GBIF Geocoder. This file is intended to help both humans and AI assistants quickly understand the project.

## Project overview

A client-side geocoder that converts locality strings (e.g. "Cambodia: Ratanakiri Province, Virachey National Park") into geographic coordinates using [GBIF](https://www.gbif.org) occurrence data. Results are scored by confidence, displayed on an interactive map, and available as GeoJSON.

- **Live demo:** https://rdmpage.github.io/gbif-geocoder/
- **Architecture:** Single-page app, everything in `index.html` (~850 lines of HTML + CSS + JS)
- **No server required** -- runs entirely in the browser on GitHub Pages
- **No build step** -- no npm, no bundler, no transpilation

## Repository files

| File | Purpose |
|------|---------|
| `index.html` | The entire application: UI, inline CSS, scoring logic, GBIF API calls, map rendering |
| `leaflet.latlng-graticule.js` | Leaflet plugin for lat/lng grid lines (from [cloudybay/leaflet.latlng-graticule](https://github.com/cloudybay/leaflet.latlng-graticule), unmodified) |
| `README.md` | User-facing documentation with usage examples and scoring explanation |
| `Notes.md` | Historical development notes, test cases, and known issues |
| `CLAUDE.md` | This file -- project architecture and design decisions |
| `.gitignore` | Excludes legacy server-side files from the repo |

**Legacy files (gitignored):** The app was originally a Glitch-hosted Node.js/Express app. Files like `server.js`, `routes.js`, `package.json`, `views/`, `hull.js`, and `wicket.js` exist locally but are excluded from the repo. They are not used by the current client-side app.

## External dependencies

Both loaded from CDN, no local copies:

| Library | Version | Purpose |
|---------|---------|---------|
| [Leaflet](https://leafletjs.com/) | 1.0.3 | Map rendering, markers, popups, GeoJSON layers |
| [Turf.js](https://turfjs.org/) | 4.7.3 | Geographic computations: `centerOfMass`, `distance`, `circle`, `bbox` |

## Data flow

```
User enters query (textarea + Search button, or ?q= URL parameter)
  |
  v
go() -- validate input, update URL via history.replaceState()
  |
  v
search_gbif(query)
  |-- clean_string(): strip punctuation, HTML tags, latinize accents, lowercase
  |-- fetch GBIF API: /v1/occurrence/search?q=...&hasCoordinate=true&hasGeospatialIssue=false&limit=10
  |-- For each of the up to 10 results:
  |     |-- computeConfidence(queryWords, result): score the match
  |     |-- buildLabel(result): construct geographic hierarchy string
  |     |-- findMostSpecificName(result): extract primary place name
  |     |-- Create Pelias-style GeoJSON Feature with properties
  |-- Sort features by confidence descending
  |-- Filter out low-confidence outliers (confidence < bestConfidence * 0.65)
  |-- Compute bounding box from remaining point coordinates
  |-- Compute bounding circle: turf.centerOfMass() + turf.distance() + turf.circle()
  |-- Update bbox to encompass full circle via turf.bbox(circle)
  |-- Return GeoJSON FeatureCollection
  |
  v
show_map(query) -- display results
  |-- add_data(geojson): render markers on Leaflet map, fit bounds with 30px padding
  |-- Create Blob URL for GeoJSON export (excludes _visual features)
  |-- Show "Copy link" and "Export as GeoJSON" in info bar
```

**Alternative output:** If URL has `&format=json`, `search_json()` is called instead of `show_map()`. It renders raw GeoJSON in a `<pre>` tag (API mode).

## Scoring algorithm

The `computeConfidence(queryWords, result)` function scores each GBIF result against the query. Three components are blended:

| Component | Weight | What it measures |
|-----------|--------|-----------------|
| **Coordination** | 50% | Fraction of query terms that matched at least one geographic field |
| **Specificity** | 30% | Weighted by geographic level -- locality-level matches (weight 1.0) score higher than continent-level (0.1) |
| **Jaccard** | 20% | Penalises results with many unmatched extra geographic terms |

**Formula:** `confidence = 0.5 * coordination + 0.3 * specificity + 0.2 * jaccard`, clamped to [0, 1], rounded to 3 decimal places.

### Geographic fields scored

Each GBIF result field is assigned a specificity weight. Higher weight = more specific = better match:

| GBIF field | Weight | Output layer |
|------------|--------|-------------|
| `continent` | 0.1 | continent |
| `country` | 0.2 | country |
| `waterBody` | 0.5 | water_body |
| `islandGroup` | 0.5 | island_group |
| `stateProvince` | 0.5 | region |
| `island` | 0.6 | island |
| `county` | 0.6 | county |
| `municipality` | 0.7 | locality |
| `locality` | 1.0 | locality |
| `verbatimLocality` | 1.0 | locality |

### Fuzzy matching

- **Algorithm:** Levenshtein edit distance (dynamic programming implementation)
- **Similarity:** `1.0 - (editDistance(a, b) / max(len(a), len(b)))`
- **Threshold:** 0.7 -- words below this similarity are not counted as matches
- **Stop words removed before matching:** `la, le, les, el, de, del, des, from, in, of, the, and, or, a, an, &`
- **Text preprocessing:** Accented characters latinized to ASCII, punctuation stripped, lowercased

## Key design decisions

1. **`verbatimLocality` included at weight 1.0** -- Many GBIF records have location detail only in `verbatimLocality` while structured fields like `locality`, `island`, `islandGroup` are empty. Without scoring it, queries like "little barrier island auckland" scored ~0.2 instead of 0.9.

2. **GBIF API `limit=10`** -- Balance between coverage and performance. Enough to find good matches without overwhelming the map or causing slow responses.

3. **Levenshtein for fuzzy matching** -- Handles spelling variations common in geographic names. Simpler and more predictable than n-gram or phonetic approaches. Threshold 0.7 avoids false positives while allowing minor differences.

4. **Coordination weighted highest (50%)** -- The most important signal is whether query terms are actually present in the result. Prevents matching unrelated areas that happen to share one term.

5. **Client-side GBIF API calls** -- GBIF's occurrence API supports CORS, so no server proxy is needed. This means zero server costs and simple static hosting on GitHub Pages.

6. **Bounding circle as visual aid** -- Computed from `turf.centerOfMass` + max distance to any point. Marked with `_visual: true` property and excluded from GeoJSON export. Map bounds are fit to the circle's bbox (not just the points) so the circle is fully visible.

7. **Pelias-style GeoJSON output** -- Following the [Pelias geocoder](https://pelias.io/) response format makes the output familiar and interoperable with other geocoding tools.

8. **All code in one file** -- Keeps the project simple and deployable as a single HTML file. No module system, no imports, no build step.

9. **Low-confidence filtering** -- After scoring, results with confidence below 65% of the best result's confidence are filtered out before computing the bounding circle. This prevents low-scoring outliers (e.g. a state-level match when the best result is a precise locality match) from inflating the circle and pulling the map view away from the best results. The threshold is relative rather than absolute so that vague queries still show results.

## URL parameters

| Parameter | Example | Behaviour |
|-----------|---------|-----------|
| `q` | `?q=Cambodia%3A%20Ratanakiri` | Pre-fills search box and auto-runs search on page load |
| `format` | `?q=...&format=json` | Returns raw GeoJSON in `<pre>` instead of rendering the map |

The URL bar is updated via `history.replaceState()` on each search so the current query is always shareable without cluttering browser history.

## CSS architecture

All styles are inline in a `<style>` block within `index.html`:

- **Layout:** Flexbox column -- header, description bar, info bar, map (flex: 1 fills remaining space)
- **Dark mode:** `@media (prefers-color-scheme: dark)` with dark navy backgrounds (`#1a1a2e`, `#16213e`), blue accents (`#4a9eed`), light text (`#e0e0e0`)
- **Mobile responsive:** `@media (max-width: 600px)` hides the title, makes search box full-width, reduces font sizes, wraps info bar
- **Marker styling:** `.mydivicon` has transparent background/border; each marker is an inline `<div>` circle sized and coloured by confidence

## Marker rendering

Markers are Leaflet `divIcon` elements scaled by confidence:

- **Size:** `10 + 14 * confidence` pixels (range: 10px at confidence 0 to 24px at confidence 1)
- **Opacity:** `0.55 + 0.45 * confidence` (range: 0.55 to 1.0)
- **Colour:** `#1b6ec2` (blue), consistent in light and dark mode
- **Popups** (on hover/click) show: label, geographic layer, confidence score, decimal coordinates, DMS coordinates, matched terms, GBIF occurrence link, coordinate uncertainty

## Functions reference

| Function | Purpose |
|----------|---------|
| `latinize(str)` | Convert accented Unicode characters to ASCII equivalents |
| `clean_string(s)` | Remove punctuation, HTML tags, extra spaces; latinize and lowercase |
| `uniqueArray(arr)` | Remove duplicates and empty strings from an array |
| `removeStopWords(arr)` | Filter out common geographic stop words |
| `editDistance(a, b)` | Levenshtein edit distance via dynamic programming |
| `wordSimilarity(a, b)` | Normalised fuzzy similarity [0, 1] based on edit distance |
| `computeConfidence(queryWords, result)` | Main scoring function: coordination + specificity + Jaccard |
| `makeIcon(confidence)` | Create Leaflet divIcon sized and styled by confidence |
| `toDMS(dd, isLat)` | Convert decimal degrees to degrees/minutes/seconds string |
| `buildPopup(feature)` | Generate HTML popup content for a map marker |
| `onEachFeature(feature, layer)` | Leaflet callback to attach popup to each marker |
| `create_map()` | Initialise Leaflet map with OSM tiles and graticule overlay |
| `clear_map()` | Remove existing GeoJSON layer from map |
| `add_data(data)` | Add GeoJSON features to map with confidence-scaled icons, fit bounds |
| `buildLabel(result)` | Construct comma-separated geographic hierarchy label |
| `findMostSpecificName(result)` | Extract the most specific geographic name available |
| `search_gbif(search)` | Fetch GBIF API, score results, build GeoJSON FeatureCollection |
| `show_map(q)` | Orchestrate search and map display, update status bar and export links |
| `copyShareLink(el)` | Copy current query URL to clipboard (with execCommand fallback) |
| `go()` | Main event handler: validate input, update URL, trigger search |
| `search_json(q)` | JSON API mode: render raw GeoJSON in `<pre>` tag |
| `getUrlParam(name)` | Extract a URL query parameter by name |
