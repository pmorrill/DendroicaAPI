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

> required: token

Response Object:

|Field|Type|Description|
|-----|----|-----------|
|token|String|The token to use in subsequent calls for this user|

Other fields returned are described in the following entry point response object.


### User Profile ###

> /api/query/profile

Parameters:
> required: token
> 
> optional: projectId

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
|speciesNaming|String|The preferred naming convnetion (commonName\|scientificName)|
|speciesSortBy|String|The preferred sorting field (alphabetic\|checklistDefault|)|
|lang|String|The preferred language|
|excludeNonBreeding|String|Only present if projectId provided: wheter to exclude non-breeding species in regions|
|excludeRare|String|Only present if projectId provided: whether to exclude rare species regions|
|checklist|String|Only present if projectId provided: the checklist that will be used to deliver species names|



## Error Responses ##

In the evennt of an error in the API, the following JSON structure will be returned:

|Field|Type|Description|
|-----|----|-----------|
|errorCode|Integer|A numeric code|
|errorMsg|String|A brief explanation of the errors|

A list of current error codes / messages can be pulled from here:

> /api/errorCodes


## Data Querying ##

All results are returned as 'data frames', suitable for R client parsing / management.




### Projects ###

Return a list of projects that allow public participation, along with protocols

> /api/projects



### Project Regions ###

> /api/projectRegions


### Project Species ###

> /api/query/species


### Species Images ###

> /api/speciesImages


### Species Sounds ###

> /api/speciesSounds


### Species Maps ###


> /api/speciesMaps


### Species Regions ###

> /api/query/speciesRegions


### File Regions ###

> /api/fileRegions


