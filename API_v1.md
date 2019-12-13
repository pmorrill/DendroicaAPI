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
|speciesSortBy|String|The preferred sorting field (alphabetic\|checklistDefault|)|
|lang|String|The preferred language (en\|fr\|es)|
|excludeNonBreeding|String|Only present if projectId provided: whether to exclude non-breeding species in regions (Y\|N)|
|excludeRare|String|Only present if projectId provided: whether to exclude rare species regions (Y\|N)|
|checklistId|Integer|Only present if projectId provided: the checklist id that will be used to deliver species names (not yet implemented)|
|checklistName|String|Only present if projectId provided: the checklist name that will be used to deliver species names (not yet implemented)|

Note that checklist name might be the same as the defaultChecklist field in the /projects
entrypoint described below. But if it is different, we **may** use that preferred checklist to
deliver species taxonomy, rather than the default checklist for the project.



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
|name|String|Projetc name|
|description|String|Project description|
|masterRegionId|Integer|The region that defines the goepolitical boundary of the project|
|defaultCheckList|String|The default checklist for the project|


### Project Regions ###

> /api/projectRegions

|Field|Type|Description|
|-----|----|-----------|


### Project Species ###

> /api/query/species


|Field|Type|Description|
|-----|----|-----------|


### Species Images ###

> /api/speciesImages

|Field|Type|Description|
|-----|----|-----------|


### Species Sounds ###

> /api/speciesSounds

|Field|Type|Description|
|-----|----|-----------|


### Species Maps ###


> /api/speciesMaps

|Field|Type|Description|
|-----|----|-----------|

### Species Regions ###

> /api/query/speciesRegions

|Field|Type|Description|
|-----|----|-----------|


### File Regions ###

> /api/fileRegions

|Field|Type|Description|
|-----|----|-----------|
