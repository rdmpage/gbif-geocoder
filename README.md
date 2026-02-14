# GBIF georeferencing


## Idea

Do simple GBIF search on geographic string, use that to georeference string

Could have a "blacklist to remove/ignore obviously wrong records

Could add more data, e.g. already georeferenced GenBank sequences to build up database

Need WKT to store polygon in Darwin Core, use http://dev.openlayers.org/examples/vector-formats.html for testing, need wicket to do conversion https://www.npmjs.com/package/wicket

- Need to add known locality as parameter so we can test (e.g., using already geocoded samples)
- Have "blacklist" as a parameter, so can exclude GBIF records that are known to be problematic
- Rethink interface
- Maybe use http://pelias.io/ as model for API output
- See for lat lng graticule https://github.com/cloudybay/leaflet.latlng-graticule/blob/master/leaflet.latlng-graticule.js
- Could store results in Darwin Core fields in JSON result

### Search scoring

See https://www.elastic.co/guide/en/elasticsearch/guide/current/practical-scoring-function.html for notes on query coordination and scoring, could use this idea.

### Example of polygon in GBIF

See https://www.gbif.org/occurrence/1675326320  with 

```Footprint WKT	POLYGON ((167.396375822793 -21.1774610538693,167.396203806315 -21.1593971462459,167.415460744443 -21.159234694187,167.415635133176 -21.1772985818478,167.396375822793 -21.1774610538693))```

Cyrtandra mareensis Däniker


## Extract localities from BHL text

Use succint data structures http://stevehanov.ca/blog/index.php?id=120 and list of countries and admin level 1 localities from geonames to find locations, grab surrounding text, georeference, store centroid and polygon in database, index using Elastic search.


## Places to add

Viet Nam:Kon Tum, Ngoc Linh Nature Reserve

SEE also https://mapzen.com/blog/mapzen-places

## Occurrences to blacklist

### https://www.gbif.org/occurrence/1305957572

Has locality info (Gazi Creek in the Gazi Bay) but "only georeferenced to country level"

###

Search for "SUMATRA, Aceh, Mont Leuser Nat. Park, Ketambe" returns two occurrences, https://www.gbif.org/occurrence/1230064953 is clearly wrong. Note "Georeference Verification Status	requires verification"


## GenBank already in GBIF via Plazi

### Pseudopoda

https://www.ncbi.nlm.nih.gov/nuccore/1216584620 as "China", googling "One new Pseudopoda species group (Araneae: Sparassidae) from Yunnan
            Province, China, with description of three new species" gives paper 
"Bai Autonomous Prefecture of Dali, Eryuan County, Mt. Yueling" from Zootaxa paper https://www.researchgate.net/publication/319566545_One_new_Pseudopoda_species_group_Araneae_Sparassidae_from_Yunnan_Province_China_with_description_of_three_new_species one locatlity is from Plazi 
https://www.gbif.org/occurrence/1612095120

## Good examples

https://www.ncbi.nlm.nih.gov/protein/1243020278

Colombia: Meta, Villavicencio, Barrio Vanguardia, carretera Vieja Villavicencio-Restrepo, km 2

Cambodia: Ratanakiri Province, Virachey National
                     Park
                     
                     http://onlinelibrary.wiley.com/doi/10.1111/j.1096-3642.2004.00125.x/full
                     
 ### GenBank
 
 https://www.ncbi.nlm.nih.gov/nuccore/KY885060.1 "Colombia: Antioquia, Dabeiba, Chichirido" not georef, but paper online  https://www.researchgate.net/publication/318323147_A_new_species_of_Andinobates_Anura_Dendrobatidae_from_the_Uraba_region_of_Colombia and in Plazi https://www.gbif.org/occurrence/search?dataset_key=48f25ad8-5841-4eab-a32a-f03853045968
 
 ### GenBank
 
 Ecuador: Galapagos, Isabela, Tagus Cove https://www.ncbi.nlm.nih.gov/nuccore/JX516097.1
 
 ### BioStor
 
 http://direct.biostor.org/reference/55073.text
 
 Psephenops lupita no records in GBIF, locality in text Xico Viejo Village 
(1,800 m altitude). Municipality of Xico, 
Veracruz State, Mexico


## Supplementary info

https://www.ncbi.nlm.nih.gov/pubmed/28611400 https://www.ncbi.nlm.nih.gov/nuccore/KY617497.1


## Test examples


## Bad examples


### Australia, Curramore Road, C.15 KM NW of Maleny, Queensland

Note that GBIF strings include stuff like https://gbif.org/occurrence/332017393 15 KM NW OF MALENY; CO-ORDINATES APPROXIMATE so may need to do some cleaning

also https://gbif.org/occurrence/1257466377 has "not applicable"

### Trinidad and Tobago: Tobago Island, Chat-Lotteville

### Rangitoto Island (all over the place)


https://www.ncbi.nlm.nih.gov/nuccore/KY748110.1  Madagascar: Sorata Picks up S. America, and also the GBIF records in Madagascar are from Plazi and are broken https://www.gbif.org/occurrence/1563404813 and https://www.gbif.org/occurrence/1563404817

### Viet Nam: Cuc Phuong National Park


Naturalis specimens from this park are in the wrong place https://protectedplanet.net/7878


### Bukit Sindap, Hose Mountains

Two records, https://gbif.org/occurrence/575213114 and https://gbif.org/occurrence/1140742175, differ in sign of latitude :(

### Loch Lommond

Matches Scotland and Tasmania, and circle bounding the points looks horrible!


## References

John Wieczorek , Qinghua Guo & Robert Hijmans (2004) The point-radius method for georeferencing locality descriptions and calculating associated uncertainty, International Journal of Geographical Information Science, 18:8, 745-767, DOI: 10.1080/13658810412331280211
