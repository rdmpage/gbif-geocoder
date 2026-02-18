# GBIF Geocoder

[![DOI](https://zenodo.org/badge/1158146924.svg)](https://doi.org/10.5281/zenodo.18678262)

Geocode locality strings using [GBIF](https://www.gbif.org) occurrence data.

**Live demo:** https://rdmpage.github.io/gbif-geocoder/

## How it works

Enter a locality string such as `Cambodia: Ratanakiri Province, Virachey National Park` and the geocoder searches the GBIF occurrence API for georeferenced records whose geographic metadata matches the query. Results are scored, ranked by confidence, and displayed on a map with a bounding circle.

Results are also available as a [Pelias](https://pelias.io/)-style GeoJSON FeatureCollection. Each feature includes the confidence score, matched terms, geographic layer (e.g. locality, region, country), coordinate uncertainty, and a link to the source GBIF occurrence record.

## Confidence scoring

Each GBIF result is scored against the query using three components:

| Component | Weight | Description |
|---|---|---|
| **Query coordination** | 50% | Fraction of query terms that matched at least one geographic field in the GBIF record. |
| **Specificity** | 30% | Matches at more specific geographic levels score higher. Locality = 1.0, municipality = 0.7, county = 0.6, state/province = 0.5, country = 0.2, continent = 0.1. |
| **Jaccard similarity** | 20% | Penalises results that have many unmatched extra geographic terms relative to the query. |

Query terms are matched against GBIF geographic fields (continent, country, stateProvince, county, municipality, locality, waterBody, island, islandGroup) using fuzzy matching based on Levenshtein edit distance. Words with similarity below 0.7 are not counted as matches. Common stop words (the, de, la, of, etc.) are removed before scoring.

## GeoJSON output

Results follow the Pelias geocoder response format:

```json
{
  "geocoding": { "query": "...", "version": "0.2" },
  "type": "FeatureCollection",
  "bbox": [minLon, minLat, maxLon, maxLat],
  "features": [
    {
      "type": "Feature",
      "geometry": { "type": "Point", "coordinates": [lon, lat] },
      "properties": {
        "source": "gbif",
        "source_id": "5106764313",
        "name": "Virachey National Park",
        "label": "Ratanakiri Province, Virachey National Park, Cambodia, ASIA",
        "confidence": 0.88,
        "layer": "locality",
        "matched_terms": ["cambodia", "ratanakiri", "province", "virachey", "national", "park"],
        "country": "Cambodia",
        "region": "Ratanakiri",
        "coordinate_uncertainty_m": 100
      }
    }
  ]
}
```

## API usage

Add `&format=json` to the URL to get raw GeoJSON instead of the map interface:

```
https://rdmpage.github.io/gbif-geocoder/?q=Cambodia:%20Ratanakiri%20Province&format=json
```

## Built with

- [Leaflet](https://leafletjs.com/) for the map
- [Turf.js](https://turfjs.org/) for geographic computations
- [GBIF Occurrence API](https://www.gbif.org/developer/occurrence) for data
- No server required -- runs entirely in the browser on GitHub Pages
