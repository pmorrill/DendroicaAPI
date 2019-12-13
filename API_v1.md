# Dendroica Mobile App API Verson 1.0 #

This set of entrypoints is designed to serve the needs of Dendroica mobile app.

All queries are over HTTPS GET requests. Query responses are JSON objects, mostl structured as a 'data frame': attribute names are the data
field names, and attribute values are vectors of field values.

## API Version ##


> /api/info

Parameters: none

|Field|Type|Description|
|-----|----|-----------|
|apiVersion|Float|A version nunber for the API|
|dataVersion|Float|A version nunber for the data|
|serverTimeStamp|Long|The current server time as UNIX timestamp|

The `apiVersion` field will not change unless we upgrade the communicaton protocol, which would mean work in the API and also the app.
Therefore, a check of this value against a hard-coded vaue in the app is sufficient.

The `dataVersion` field may increment, and this would signal that the app should perform a complete refresh of its
data tables (ie: not use the `ifModifiedSince` parameter in data queries).

The `serverTimeStamp` field can be stored for later use in data queries, as the `ifModifiedSince` parameter.

## Authentication ##

> /api/authenticate?username=asdf&password=qwerty

Parameters:

> required String: username
>
> required String: password

Response Object:

|Field|Type|Description|
|-----|----|-----------|
|token|String|The token to use in subsequent calls for this user|

The token lifespan is initially set to 24 hours, however it is extended accordingly on each use. It
will therefore always be 24 hours from the time of the most recent transaction
with the API server.

Other fields returned are described in the following entry point response object.


### User Profile ###

> /api/query/profile

Parameters:
> required String: token
> 
> optional Integer: projectId

Response Object:

|Field|Type|Description|
|-----|----|-----------|
|firstname|String|The user's firstname|
|lastname|String|The user's surname|
|email|String|The user's email addresss|
|username|String|The user's username|
|userId|Integer|The user's unique id|
|preferences|JSON Object|The user's preferences (see below)|

#### User Preferences ####

|Field|Type|Description|
|-----|----|-----------|
|speciesNaming|String|The preferred naming convention (commonName\|scientificName)|
|speciesSortBy|String|The preferred sorting field (alphabetic\|checklistDefault)|
|lang|String|The preferred language (en\|fr\|es)|
|excludeNonBreeding|String|**Only present if projectId provided:** whether to exclude non-breeding species in regions (Y\|N)|
|excludeRare|String|**Only present if projectId provided:** whether to exclude rare species regions (Y\|N)|
|checklistId|Integer|**Only present if projectId provided:** the checklist id that will be used to deliver species names (not yet implemented)|
|checklistName|String|**Only present if projectId provided:** the checklist name that will be used to deliver species names (not yet implemented)|

Note that checklist name might be the same as the defaultChecklist field in the /projects
entrypoint described below. But if it is different, we **may** use that preferred checklist to
deliver species taxonomy, rather than the default checklist for the project.

Note that if the user's language prefernce is 'es' (Spanish), the interface should display scientific names, even if the 
user's naming preference is `commonName`. We do not have Spanish common names in our system.


## Error Responses ##

In the event of an error in the API, the following JSON structure will be returned:

|Field|Type|Description|
|-----|----|-----------|
|errorCode|Integer|A numeric code|
|errorMsg|String|A brief explanation of the errors|

A list of current error codes / messages can be pulled from here:

> /api/errorCodes


## Data Querying ##


All data queries require a token parameter, returned earlier by the Authentication entrypoint.

All results are returned as JSON objects, suitable for processing as 'data frames'.




### Projects ###

Return a list of Dendroica projects.

> /api/projects

|Field|Type|Description|
|-----|----|-----------|
|id|Integer|Project id for use in subsequent calls|
|name|String|Project name|
|description|String|Project description|
|masterRegionId|Integer|The region that defines the goepolitical boundary of the project|
|defaultCheckList|String|The default checklist for the project|


### Project Regions ###

> /api/projectRegions

|Field|Type|Description|
|-----|----|-----------|
|id|Integer|Region id|
|parentRegionId|Integer|If the region is a sub-region (e.g.: state/province within a country), this points to the parent region|
|region|String|Region name|
|abbrev|String|Abreviation for the region|


### Project Species ###

> /api/query/species


|Field|Type|Description|
|-----|----|-----------|
|id|Integer|Species id|
|commonName|String|The species common name under the current taxonomic checklist|
|scientificName|String|The species scientific name under the current taxonomic checklist|
|mapDescription|String|A text description of the species' range|
|songDescription|String|A text description of the species' song|
|checklistOrder|Integer|The species ordering under the current taxonomic checklist|

The species should be presented in the order of the `checklistOrder` field, unless the user's
preferences specify that the ordering should be alphabetical.

Note that if the user's language prefernce is 'es' (Spanish), the interface should display scientific names, even if the 
user's naming preference is `commonName`. We do not have Spanish common names in our system.


### Species Images ###

> /api/speciesImages

|Field|Type|Description|
|-----|----|-----------|
|id|Integer|File id|
|speciesId|Integer|The species id this file is linked to|
|url|String|The url suffix to obtain the image file from natureinstruct|
|source|String|The source of the image (credit)|
|displayOrder|Integer|The order that the image should be displayed for a given species|



### Species Sounds ###

> /api/speciesSounds

|Field|Type|Description|
|-----|----|-----------|
|id|Integer|File id|
|speciesId|Integer|The species id this file is linked to|
|url|String|The url suffix to obtain the sound file from natureinstruct|
|spectrogramUrl|String|The url suffix to obtain the sound file spectrogram from natureinstruct|
|source|String|The source of the recording (credit)|
|displayOrder|Integer|The order that the recording should be displayed for a given species|

Note that the `spectrogramUrl` values may include locations that return a 404 error, as we do not have
spectrograms for every recording.


### Species Maps ###


> /api/speciesMaps

|Field|Type|Description|
|-----|----|-----------|
|id|Integer|File id|
|speciesId|Integer|The species id this file is linked to|
|url|String|The url suffix to obtain the map file from natureinstruct|
|source|String|The source of the map (credit)|
|description|String|A description of the map (North\|Central\|South\|Americas)|
|displayOrder|Integer|The order that the map should be displayed for a given species|



### Species Regions ###

> /api/query/speciesRegions

|Field|Type|Description|
|-----|----|-----------|
|speciesId|Integer|The species id|
|regionId|Integer|The region id|
|nonBreederInRegion|Integer|Whether the species is a non-breeder in the region (0\|1)|
|rareInRegion|Integer|Whether the species is rare in the region (0\|1)|


### File Regions ###

> /api/fileRegions

|Field|Type|Description|
|-----|----|-----------|
|fileId|Integer|The file id|
|regionId|Integer|The region id|
|displayOrder|Integer|The order that should be used to diaply the file when viewing this region|

Note that `fileId` is unique over all media files (images, sounds, and maps). However, you will never find
map file id's in the region associations: all species maps should be displayed for a given species
regardless of region.

For a given species and region, there may be no records in the `fileRegions` data set. If that is so, 
you should look to see if the region has a `parentRegionId`, and check for `fileRegions` associations with 
that parent region.

If no region-specific associations are found, then the file `displyOrder` can be used from the `speciesImages`
and / or `speciesSounds` data.

If a `fileRegion` association has been found, you should:

1. Only display files that occur in the association
2. Display those files using the displayOrder in the association

It is also important to note that a `fileRegions` association may be present for **images** and not for **sounds**
(for a given species / region). In that case, the default sounds ordering would be used, while the `fileRegion` ordering
would be used for the images. And vice-versa.....



