## ZipCodeUseCase Guide

This guide shows how to initialize and use every public method in `ZipCodeUseCase`, with suspend
and callback examples where available. The ZipCodeUseCase provides postal code lookup
functionality for retrieving location information based on zip codes.

### Getting an instance

```kotlin
val zipCodeService = BuzzebeesSDK.instance().zipCodeUseCase
```

---

### getZipcode

Retrieves location information based on the provided zip code.

- Request (caller-supplied)

| Field Name | Description                     | Mandatory | Data Type |
|------------|---------------------------------|-----------|-----------|
| zipCode    | Postal/ZIP code to lookup       | M         | String    |

- Response (`List<Zipcode>`) [source](../buzzebees_sdk/src/main/java/com/buzzebees/sdk/entity/zip_code/Zipcode.kt)  
  HTTP status: 200

### Zipcode Entity Fields

| Field Name          | Description                        | Data Type | JSON Field          |
|---------------------|------------------------------------|-----------|---------------------|
| cityId              | City identifier                    | String?   | CityId              |
| subDistrictCode     | Sub-district code                  | String?   | SubDistrictCode     |
| subDistrictName     | Sub-district name in Thai          | String?   | SubDistrictName     |
| subDistrictNameEN   | Sub-district name in English       | String?   | SubDistrictName_EN  |
| zipCode             | ZIP/postal code                    | String?   | ZipCode             |
| districtCode        | District code                      | String?   | DistrictCode        |
| districtName        | District name in Thai              | String?   | DistrictName        |
| districtNameEN      | District name in English           | String?   | DistrictName_EN     |
| provinceCode        | Province code                      | String?   | ProvinceCode        |
| provinceName        | Province name in Thai              | String?   | ProvinceName        |
| provinceNameEN      | Province name in English           | String?   | ProvinceName_EN     |
| active              | Active status flag                 | Boolean?  | Active              |
| createDate          | Creation date timestamp            | Long?     | CreateDate          |
| createBy            | Created by user                    | String?   | CreateBy            |
| modifyDate          | Modification date timestamp        | Long?     | ModifyDate          |
| modifyBy            | Modified by user                   | String?   | ModifyBy            |

- Usage

```kotlin
// Suspend
val result = zipCodeService.getZipcode("10110")

// Callback
zipCodeService.getZipcode("10110") { result ->
    when (result) {
        is ZipCodeResult.SuccessZipCode -> {
            // Handle successful zipcode lookup
            val zipcodes = result.result

            zipcodes.forEach { zipcode ->
                val zipCode = zipcode.zipCode
                val subDistrictName = zipcode.subDistrictName
                val districtName = zipcode.districtName
                val provinceName = zipcode.provinceName
                val active = zipcode.active

                println("ZIP Code: $zipCode")
                println("Sub-district: $subDistrictName")
                println("District: $districtName")
                println("Province: $provinceName")
                println("Active: $active")
            }
        }
        is ZipCodeResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

## Summary

The ZipCodeUseCase provides postal code lookup functionality within the Buzzebees SDK. It offers methods to retrieve detailed location information including sub-district, district, and province data based on Thai postal codes, supporting both Thai and English language responses.
