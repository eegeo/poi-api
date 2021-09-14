![WRLD](images/pois.jpg)

WRLD POI REST API v1.1.0
========================

This document describes version 1.1.0 of the WRLD POI REST API for submitting and managing custom Points of Interest. The POI API is primarily intended for use with [wrld-example-app](http://github.com/wrld3d/wrld-example-app).

---

## Quick Start

1. Obtain a Developer Authentication Token by signing up at [https://www.wrld3d.com/developers/](https://www.wrld3d.com/developers/)
2. Create an Application API Key at [https://www.wrld3d.com/developers/](https://www.wrld3d.com/developers/)
3. [Create a new POI Set](#newpoiset)
4. [Associate your Application API Key with your newly created POI Set](#associate)
5. [Add POIs to your POI Set](#createpoi)
6. Perform [Free-Text, Category or Indoor](#search) queries

---

Full POI API Specification
==========================

#### Authentication Token

Obtain a Developer Authentication Token by signing up at [https://www.wrld3d.com/developers/](https://www.wrld3d.com/developers/). The Developer Authentication Token allows you to make authenticated requests against the POI service.

The Authentication Token will be referred to as ```dev_auth_token``` throughout this README

#### Application API Key

The Application API Key is the API key used to authenticate the WRLD SDK. Application Keys can be associated with POI sets, providing that application access to the POI set. Obtain Application API keys from [https://www.wrld3d.com/developers/](https://www.wrld3d.com/developers/).

Application API Keys will be referred to as ```app_api_key``` throughout this README

#### POI Sets

A POI Set is a collection of Points of Interest. You can associate multiple Application API keys with POI Sets, allowing multiple applications or platforms access to the same POI Set.

A POI Set is a JSON object with the following attributes:

|Field|Type|Description|
 --- | --- | ---
|`id`|integer| a unique identifier for the POI set, this is generated for you
|`name`|string| an appropriate name for the POI set
|`api_key_permissions`|string list| app api keys that have access to this POI set

##### <a name="newpoiset"></a>Create a POI Set

To create a new POI Set, make a RESTful request passing the name of the POI set:

```sh
$ curl -v -XPOST https://poi.wrld3d.com/v1.1/poisets/?token=dev_auth_token -d '{"name":"my-poi-set"}'
```

The response will be a JSON object specifying the newly created POI Set:

```json
{
"id":1, 
"name":"my-poi-set", 
"api_key_permissions":[]
}
```

Make note of the POI Set ID, in this case ```1``` as this is used to make future requests against this newly created POI Set.

##### Query all POI sets

All POI Sets you own can be listed by making a RESTful request to the poisets resource:

```
$ curl -v https://poi.wrld3d.com/v1.1/poisets/?token=<dev_auth_token>
```

This will return a collection of all POI Sets you own

##### Query a POI set

To query an individual POI Set, make a RESTful request to the poiset:

```
$ curl -v https://poi.wrld3d.com/v1.1/poisets/<SID>?token=<dev_auth_token>
```

Where ```<SID>``` is the POI Set ID to query. This will return the POI Set as JSON.

##### Delete a POI Set

To delete a POI Set, make a RESTful request to the poiset:
```
$ curl -v -XDELETE https://poi.wrld3d.com/v1.1/poisets/<SID>?token=<dev_auth_token>
```
Where ```<SID>``` is the POI Set ID to delete.

##### <a name="associate"></a>Associate App API Key

Application API Keys can be associated with a POI Set to provide that Application access to the POIs in the POI Set.

To do this, make a RESTful call:

```sh
$ curl -v -XPOST https://poi.wrld3d.com/v1.1/poisets/<SID>?token=<dev_auth_token> -d '{"apikey":"<app_api_key>"}'
```

Where ```app_api_key``` is the Application API Key, and where ```<SID>``` is the POI Set ID to add the Application API Key to.

##### Unassociate App API Key

To unassociate an Application API Key from a POI Set, perform the following RESTful call:

```sh
$ curl -v -XDELETE https://poi.wrld3d.com/v1.1/poisets/<SID>/<app_api_key>?token=<dev_auth_token>
```

Where ```app_api_key``` is the Application API Key, and where ```<SID>``` is the POI Set ID to remove the Application API Key from.

---
Points of Interest
==================

#### POI

A POI is a Point of Interest. POIs are added to POI Sets and can be searched by applications with access to that POI Set.

The default POI and default taxonomy is supported by [wrld-example-app](http://github.com/wrld3d/wrld-example-app) - however both the POI API and wrld-example-app can be customised, so if the default behaviour does not meet your needs you can change the behaviour to suit.

A POI is a JSON object with the following attributes:

|Field|Required|Empty|Type|Description|
 --- | --- | --- | --- | ---
|`id`|generated|no|integer| a unique identifier for the POI, this is generated for you
|`title`|required|no|string| an appropriate title for the POI (i.e. place name)
|`subtitle`|required|yes|string| an appropriate subtitle for the POI (i.e. address)
|`tags`|required|no|string| a whitespace separated list of [tags](#tags)
|`lat`|required|no|decimal| wgs84 decimal degrees latitude
|`lon`|required|no|decimal| wgs84 decimal degrees longitude
|`height_offset`|optional|no|decimal| height in metres of POI above ground level (default: 0)
|`indoor`|optional|no|boolean| whether POI is indoor (default: false)
|`indoor_id`|optional|yes|string| the id of the indoor map the POI belongs to (default: empty)
|`floor_id`|optional|no|integer| the floor_id of the indoor map the POI belongs to (default: 0)
|`user_data`|optional|yes|json| a json object of custom user data (default: empty)

The ```user_data``` attribute is entirely custom, however in [wrld-example-app](http://github.com/wrld3d/wrld-example-app) the following default ```user_data``` attributes are supported:

|Field|Type|Description|
 --- | --- | ---
|`image_url`|string| a url to an image to display for the POI
|`phone`|string| a phone number for the POI
|`web`|string| a website for the POI
|`description`|string| a description for the POI
|`highlight`|string \| string[]| id(s) of the highlight(s) to enable for a POI
|`highlight_color`|int[]| highlight color defined as an [R,G,B,A] int array (0-255)
|`highlight_border_thickness`|decimal| a value between 0 and 1 that describes how thick the border for area highlights should be (default: 0.5)
|`custom_view`|string| a url of a html view
|`custom_view_height`|int| define the height of the custom_view (optional)

##### <a name="createpoi"></a>Create a new outdoor POI

To create a new POI, make the following RESTful call:

```sh
$ curl -v -XPOST https://poi.wrld3d.com/v1.1/poisets/<SID>/pois/?token=<dev_auth_token> -d '{
  "title":"WRLD Dundee",
  "subtitle":"Suite 2, Westport House",
  "tags":"office business",
  "lat":56.459937,
  "lon":-2.978124
}'
```

Where ```<SID>``` is the POI Set ID to add the POI to. 

##### Update an existing POI

To update attributes of an existing POI, make the following RESTful call:

```sh
$ curl -v -XPUT https://poi.wrld3d.com/v1.1/poisets/<SID>/pois/<PID>?token=<dev_auth_token> -d '{
  "title":"A new Title",
  "subtitle":"A new Subtitle"
}'
```

Where ```<SID>``` is the POI Set ID the POI belongs to, and ```<PID>``` is the ID of the POI

##### Deleting POIs

To delete an existing POIs in a POI set, make the following RESTful call:

```sh
$ curl -v -XDELETE https://poi.wrld3d.com/v1.1/poisets/<SID>/pois/<PID>?token=<dev_auth_token>
```

Where ```<SID>``` is the POI Set ID the POI belongs to, and ```<PID>``` is the ID of the POI

To delete all POIs in a POI set, make the following RESTful call:

```sh
$ curl -v -XDELETE https://poi.wrld3d.com/v1.1/poisets/<SID>/pois/?token=<dev_auth_token>
```

Where ```<SID>``` is the POI Set ID the POIs belong to.

##### Querying POIs

To query POIs in a POI set, make the following RESTful calls:

```sh
$ curl -v https://poi.wrld3d.com/v1.1/poisets/<SID>/pois/?token=<dev_auth_token>
$ curl -v https://poi.wrld3d.com/v1.1/poisets/<SID>/pois/<PID>?token=<dev_auth_token>
```

Where ```<SID>``` is the POI Set ID the POIs belong to, and ```<PID>``` is the ID of the POI

#### Indoor POI

Indoor POIs are supported by default in [wrld-example-app](http://github.com/wrld3d/wrld-example-app). To create an indoor POI three optional attributes must be defined when creating the POI:

|Field|Type|Description|
 --- | --- | ---
|`indoor`|boolean| whether POI is indoor (default: false)
|`indoor_id`|string| the id of the indoor map the POI belongs to (default: empty)
|`floor_id`|integer| the floor_id of the indoor map the POI belongs to (default: 0)

The ```indoor_id``` is the id of the indoor map. See [wrld-indoor-maps-api](http://github.com/wrld3d/wrld-indoor-maps-api) for more details.

##### Create a new indoor POI

To create a new POI, make the following RESTful call:

```sh
$ curl -v -XPOST https://poi.wrld3d.com/v1.1/poisets/<SID>/pois/?token=<dev_auth_token> -d '{
  "title":"Primark",
  "subtitle":"Overgate",
  "tags":"clothing shopping",
  "lat":56.460189,
  "lon":-2.971023,
  "indoor":true,
  "indoor_id":"overgate_dundee",
  "floor_id":0
}'
```

Where ```<SID>``` is the POI Set ID to add the POI to. 

#### Tags

The ```tags``` property of a POI is flexible. You can define your own taxonomy and categorisation to suit your need. However, the ```tags``` field in the POI must fulfil the following requirements:

1. All tags must be separated by one whitespace. Eg: ```"services office business"```
2. Each tag may only contain lowercase alpha characters and/or underscore if required. Numbers are not allowed. Eg: ```"restaurant bar food_drink"```
3. Tags are ordered from the most specific first to least specific. Eg: ```"burgers restaurant bar food_drink"```, therefore that particular POI is best described as a __burger__ joint and least described by __food_drink__, even though all the tags are relevant to the POI

#### Default Tags

A default set of high level tags is provided in [wrld-example-app](http://github.com/wrld3d/wrld-example-app). The categories supported are:

|Category|Tags|
 --- | ---
 |Shopping|`fashion`, `fashion_kids`, `fashion_men`, `fashion_women`, `florist`, `groceries`, `night_shopping`, `technology`
 |Transport|`airport`, `bus`, `charge_point`,`cycling`, `ferry`, `fuel`, `limo`, `parking`, `subway`, `taxi`, `train`, `tram`, `transport`, `uber`
 |Indoor Entertainment|`art`, `casino`, `cinema`, `gallery`, `library`, `music`, `shopping`, `spa`
 |Outdoor Entertainment|`camping`, `fishing`, `football`, `golf`, `historic_monument`, `observation_point`, `park`, `playing_field`, `race_track`, `sailing`, `shooting`, `skiing`, `sports_field`, `sports`, `zoo`
 |Accommodation|`accommodation`, `lodge`, `home`, `home_apartment`, `hostel`, `hotel`
 |Landscape Feature|`forest`, `lake`, `mountain`, `river`
 |Religion|`christianity`, `church`, `islam`, `judaism`, `place_of_worship`
 |Bathrooms|`bathroom`, `toilet_ladies`, `toilet_men`
 |Food & Drink|`bed_breakfast`, `beer`, `burgers`, `chinese_thai`, `cocktail`, `coffee`, `donuts`, `food_drink`, `indian`, `pizza`, `seafood`, `steak`, `sushi`, `vegan`, `vegetarian`, `wine`
 |Buildings|`atm`, `bank`, `dentist`, `farm`, `fire`, `hospital`, `observatory`, `offices`, `payphone`, `pharmacy`, `police`, `post_office`, `telephone_booth`, `tourist_info`, `town_hall`, `traffic_counter`, `university`, `vet`, `wind_turbine`
 |Smart Office|`badge_reader`, `bin`, `bin_full`, `co_two_detector`, `collaboration_space`, `defibrillator`, `department`, `electricity_meter`, `elevator`, `emergency_exit`, `escalator`, `fire_extinguisher`, `first_aid`, `gas_meter`, `hvac_unit`, `meeting_room`, `meeting_room_available_soon`, `meeting_room_available`, `meeting_room_unavailable`, `occupancy_sensor`, `positioning_beacon`, `printer`, `reception`, `security_alarm`, `smart_post`, `smoke_detector`, `stairs`, `stationery`, `temperature_sensor`, `thermostat`, `video_conference`, `washing_machine`, `wi_fi_point`, `working_group`
 |Smart Home|`dining_room`, `driveway`, `fireplace`, `fridge`, `homeware`, `kitchen`, `oven`
 |Web|`click_and_collect`, `facebook`, `google_plus`, `link`, `twitter`, `picasa`, `photo`, `video`, `vimeo`, `virtual_tour`, `vr_experience`, `warning`, `wikipedia`, `youtube`
 |Other|`accessible`, `alert`, `border`, `business`, `childcare`, `digital_sign`, `education`, `entertainment`, `events`, `general`, `health`, `lifeguard`, `meter`, `message`, `mobile_phone`, `my_location`, `person`, `pin`, `remote_control`, `solar_panels`, `star`, `ticket`

---

<a name="search"></a>Searching
=========

#### Free Text Search

To perform free text search, perform the following query:

```sh
$ curl -v "https://poi.wrld3d.com/v1.1/search?s=WRLD&lat=56.460189&lon=-2.971023&apikey=<app_api_key>"
$ curl -v "https://poi.wrld3d.com/v1.1/search?s=WRLD&lat=56.460189&lon=-2.971023&r=10&apikey=<app_api_key>"
$ curl -v "https://poi.wrld3d.com/v1.1/search?s=WRLD&lat=56.460189&lon=-2.971023&n=1&apikey=<app_api_key>"
```

Where ```<app_api_key>``` is an Application API Key that has permission to access a POI Set.

Permitted arguments to the query are:

|Field|Required|Type|Description|
 --- | --- | --- | ---
|`s`|required|string| the search term
|`lat`|required|decimal| wgs84 decimal degrees latitude of the query
|`lon`|required|decimal| wgs84 decimal degrees longitude of the query
|`r`|optional|decimal| radius of the search query in metres (default: 1000.0)
|`n`|optional|integer| maximum number of results to return (default: 20)
|`ms`|optional|decimal| minimum 'score' to results. the higher the number the fewer results will be matched  (default: 0.0)
|`indoor_id`|optional|string| id of indoor map to filter results by (default: none)
|`f`|optional|integer| the floor number of the origin of the search, starting at 0 for the lowest floor (default: 0)
|`fs`|optional|integer| floor score drop off, i.e. the number of floors to search above and below (default: 15)

#####  Result Ordering and Scoring

Results are ordered by score. Scored is a measure of _closeness_ to the input query. This measurement of this closeness is complex and subject to change. But the following are factors:

1. Lexical distance from the search query
2. Distance from the query origin

#### Tags Search

To perform a tag search, perform the following query (searches for: ```"office business"```):

```sh
$ curl -v "https://poi.wrld3d.com/v1.1/tag?t=office%20business&lat=56.460189&lon=-2.971023&apikey=<app_api_key>"
```

Where ```<app_api_key>``` is an Application API Key that has permission to access a POI Set.

Permitted arguments to the query are:

|Field|Required|Type|Description|
 --- | --- | --- | ---
|`t`|optional|string| the tag(s) to search for. If not specified it will return all POIs it finds
|`lat`|required|decimal| wgs84 decimal degrees latitude of the query
|`lon`|required|decimal| wgs84 decimal degrees longitude of the query
|`r`|optional|decimal| radius of the search query in metres (default: 1000.0)
|`n`|optional|integer| maximum number of results to return (default: 20)

#### Indoor Map constrained Category Search

To perform a category search, constrained to a particular indoor map, perform the following query:

```sh
$ curl -v "https://poi.wrld3d.com/v1.1/indoor?i=<indoor_map_id>&t=office%20business&f=3&apikey=<app_api_key>"
```

Where ```<app_api_key>``` is an Application API Key that has permission to access a POI Set and ```<indoor_map_id>``` is the indoor map to constrain against.

Permitted arguments to the query are:

|Field|Required|Type|Description|
 --- | --- | --- | ---
|`i`|required|string| the indoor map ID to constrain against
|`f`|required|integer| the floor number of the origin of the search, starting at 0 for the lowest floor
|`t`|optional|string| the tag(s) to search for. If not specified it will returbn all POIs it finds
|`n`|optional|integer| maximum number of results to return (default: 20)
|`s`|optional|integer| floor score drop off, i.e. the number of floors to search above and below (default: 15)
|`lat`|optional|decimal| wgs84 decimal degrees latitude of the query, used to sort results by distance
|`lon`|optional|decimal| wgs84 decimal degrees longitude of the query, used to sort results by distance
---

#### Bulk API

It is possible to make Bulk updates to the POI API to reduce round trips. The Bulk API allows create, update and delete operations to be performed against a POI Set.

To use the Bulk API, make queries of the following form:

```sh
$ curl -v -XPOST https://poi.wrld3d.com/v1.1/poisets/<SID>/bulk/?token=<dev_auth_token> -d '{
    "create": [{ 
      "title" : "West House", 
      "subtitle" : "2 West Port, Dundee DD1 5EP", 
      "tags" : "bar food_drink",
      "lat" : 56.459336, 
      "lon" : -2.977645,
      "height_offset" : 0.1
    },
    { 
      "title" : "WRLD Dundee", 
      "subtitle" : "Westport House", 
      "tags" : "office business",
      "lat" : 56.459941, 
      "lon" : -2.978211,
      "height_offset" : 0.1,
      "indoor" : true,
      "indoor_id" : "westport_house_interior_id",
      "floor_id" : 3
    }],

    "update": [{ 
      "id" : 1,
      "title" : "West House", 
      "subtitle" : "2 West Port, Dundee DD1 5EP", 
      "tags" : "office business",
      "lat" : 56.459336, 
      "lon" : -2.977645,
      "height_offset" : 0.1
    }],

    "delete": [1,2,3,4,5]
}'
```

Where ```<SID>``` is the POI Set ID to perform the Bulk Operation on.

---

#### Disclaimer
This is a stable, semantically versioned API. 

WRLD may make changes to the API from time to time but will adhere to [Semantic Versioning](http://semver.org/) and provide backward compatible end points for as long as possible.

---

#### Contact us
If you have any problems or queries please [raise an issue](https://github.com/wrld3d/wrld-poi-api/issues/new).
