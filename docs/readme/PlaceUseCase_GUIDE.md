## PlaceUseCase Guide

This guide shows how to initialize and use every public method in `PlaceUseCase`, with suspend
and callback examples where available. The PlaceUseCase provides place and location management
functionality for retrieving place lists and detailed place information.

### Getting an instance

```kotlin
val placeService = BuzzebeesSDK.instance().placeUseCase
```

---

### getPlaceList

Retrieves a list of places based on search criteria and location filters.

- Request (caller-supplied)

| Field Name       | Description                          | Mandatory | Data Type |
|------------------|--------------------------------------|-----------|-----------|
| agencyId         | Agency identifier                    | M         | String    |
| center           | Center coordinates (lat,lng)         | M         | String    |
| distance         | Search radius in kilometers          | M         | Int       |
| top              | Maximum number of results            | M         | Int       |
| provinceCode     | Province code filter                 | O         | String?   |
| category         | Category filter                      | O         | String?   |
| mode             | Search mode                          | O         | String?   |
| requiredCampaign | Require active campaign              | O         | Boolean   |
| keyword          | Search keyword                       | O         | String?   |

- Response (`List<Place>`)
  HTTP status: 200

### Place Entity Fields

| Field Name       | Description                          | Data Type      | JSON Field        |
|------------------|--------------------------------------|----------------|-------------------|
| name             | Place name in default language       | String?        | name              |
| id               | Place unique identifier              | String?        | id                |
| location         | Geographic location coordinates      | Location?      | location          |
| isIsBuzzeBeesPlace| BuzzeBees place flag                | Boolean?       | isBuzzeBeesPlace  |
| rank             | Place ranking                        | Int?           | rank              |
| distance         | Distance from center point           | Double?        | distance          |
| buzz             | Buzz rating                          | Int?           | buzz              |
| like             | Number of likes                      | Int?           | like              |
| description      | Place description in default language| String?        | description       |
| contactNumber    | Contact phone number                 | String?        | contact_number    |
| workingDay       | Working days in default language     | String?        | working_day       |
| nameEn           | Place name in English                | String?        | name_en           |
| descriptionEn    | Place description in English         | String?        | description_en    |
| workingDayEn     | Working days in English              | String?        | working_day_en    |
| address          | Address in default language          | String?        | address           |
| addressEn        | Address in English                   | String?        | address_en        |
| imageUrl         | Place image URL                      | String?        | image_url         |
| region           | Region information                   | String?        | region            |
| category         | Place category                       | String?        | Category          |
| expert           | Expert information                   | String?        | expert            |
| services         | List of available services           | `List<Services>?`| services          |
| subDistrictCode  | Sub-district code                    | Int?           | subdistrict_code  |
| districtCode     | District code                        | Int?           | district_code     |
| provinceCode     | Province code                        | Int?           | province_code     |

### Location Entity Fields

| Field Name | Description        | Data Type | JSON Field |
|------------|-------------------|-----------|-----------|
| latitude   | Latitude coordinate| Double?   | latitude  |
| longitude  | Longitude coordinate| Double?  | longitude |

### Services Entity Fields

| Field Name | Description    | Data Type | JSON Field |
|------------|----------------|-----------|-----------|
| id         | Service ID     | Int?      | id        |
| name       | Service name   | String?   | name      |

- Usage

```kotlin
// Create form
val placeForm = PlaceForm(
    agencyId = "123",
    center = "13.7563,100.5018", // Bangkok coordinates
    distance = 10, // 10 km radius
    top = 20,
    provinceCode = "10", // Bangkok
    category = "restaurant",
    mode = "search",
    requiredCampaign = true,
    keyword = "coffee"
)

// Suspend
val result = placeService.getPlaceList(placeForm)

// Callback
placeService.getPlaceList(placeForm) { result ->
    when (result) {
        is PlaceResult.SuccessPlace -> {
            // Handle successful place list retrieval
            val places = result.result

            places.forEach { place ->
                val placeName = place.name
                val placeDistance = place.distance
                val placeAddress = place.address
                val contactNumber = place.contactNumber

                println("Place: $placeName")
                println("Distance: $placeDistance km")
                println("Address: $placeAddress")
            }
        }
        is PlaceResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### getPlaceDetail

Retrieves detailed information about a specific place.

- Request (caller-supplied)

| Field Name | Description                     | Mandatory | Data Type |
|------------|---------------------------------|-----------|-----------|
| id         | Place unique identifier         | M         | String    |
| center     | Center coordinates (lat,lng)    | M         | String    |

- Response (`Place`)
  HTTP status: 200

### Place Entity Fields

| Field Name       | Description                          | Data Type      | JSON Field        |
|------------------|--------------------------------------|----------------|-------------------|
| name             | Place name in default language       | String?        | name              |
| id               | Place unique identifier              | String?        | id                |
| location         | Geographic location coordinates      | Location?      | location          |
| isIsBuzzeBeesPlace| BuzzeBees place flag                | Boolean?       | isBuzzeBeesPlace  |
| rank             | Place ranking                        | Int?           | rank              |
| distance         | Distance from center point           | Double?        | distance          |
| buzz             | Buzz rating                          | Int?           | buzz              |
| like             | Number of likes                      | Int?           | like              |
| description      | Place description in default language| String?        | description       |
| contactNumber    | Contact phone number                 | String?        | contact_number    |
| workingDay       | Working days in default language     | String?        | working_day       |
| nameEn           | Place name in English                | String?        | name_en           |
| descriptionEn    | Place description in English         | String?        | description_en    |
| workingDayEn     | Working days in English              | String?        | working_day_en    |
| address          | Address in default language          | String?        | address           |
| addressEn        | Address in English                   | String?        | address_en        |
| imageUrl         | Place image URL                      | String?        | image_url         |
| region           | Region information                   | String?        | region            |
| category         | Place category                       | String?        | Category          |
| expert           | Expert information                   | String?        | expert            |
| services         | List of available services           | `List<Services>?`| services          |
| subDistrictCode  | Sub-district code                    | Int?           | subdistrict_code  |
| districtCode     | District code                        | Int?           | district_code     |
| provinceCode     | Province code                        | Int?           | province_code     |

- Usage

```kotlin
// Suspend
val result = placeService.getPlaceDetail(
    id = "place_12345",
    center = "13.7563,100.5018"
)

// Callback
placeService.getPlaceDetail(
    id = "place_12345",
    center = "13.7563,100.5018"
) { result ->
    when (result) {
        is PlaceResult.SuccessPlaceDetail -> {
            // Handle successful place detail retrieval
            val place = result.result

            val placeName = place.name
            val placeDescription = place.description
            val contactNumber = place.contactNumber
            val workingDay = place.workingDay
            val services = place.services

            println("Place: $placeName")
            println("Description: $placeDescription")
            println("Contact: $contactNumber")
            println("Working Days: $workingDay")

            services?.forEach { service ->
                println("Service: ${service.name}")
            }
        }
        is PlaceResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

## Summary

The PlaceUseCase provides place and location management functionality within the Buzzebees SDK. It offers methods to search for places based on location, distance, and various filters, as well as retrieve detailed information about specific places including services, contact information, and geographic data.
