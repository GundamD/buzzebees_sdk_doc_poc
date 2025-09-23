## AddressInformationUseCase Guide

This guide shows how to initialize and use every public method in `AddressInformationUseCase`, with suspend and callback examples where available. The AddressInformationUseCase provides geographic information services including provinces, districts, sub-districts, and countries data for address form population and validation.

### Getting an instance

```kotlin
val addressInformationService = BuzzebeesSDK.instance().addressInformation
```

---

### getProvinceList

Retrieves a list of all available provinces. Used for populating province dropdowns in address forms and validation.

- Request (caller-supplied)

| Field Name | Description | Mandatory | Data Type |
|------------|-------------|-----------|-----------|
| -          | None        | -         | -         |

- Response (`List<Province>`)

  HTTP status: 200

> **Province Fields Reference**
>
> See the complete Province entity fields table below for all available properties.

- Usage

```kotlin
// Suspend
val result = addressInformationService.getProvinceList()

// Callback
addressInformationService.getProvinceList { result ->
    when (result) {
        is AddressInformationResult.SuccessProvince -> {
            // Handle successful province list retrieval
            val provinces = result.result
            provinces.forEach { province ->
                val provinceCode = province.provinceCode
                val provinceName = province.provinceName
                val provinceNameEN = province.provinceNameEN
                // Populate dropdown or spinner
            }
        }
        is AddressInformationResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### getDistrictList

Retrieves a list of districts within a specific province. Used for populating district dropdowns after a province is selected.

- Request (caller-supplied)

| Field Name   | Description              | Mandatory | Data Type |
|--------------|--------------------------|-----------|-----------|
| provinceCode | Province code to filter  | M         | String    |

- Response (`List<District>`)

  HTTP status: 200

> **District Fields Reference**
>
> See the complete District entity fields table below for all available properties.

- Usage

```kotlin
// Suspend
val result = addressInformationService.getDistrictList("10") // Bangkok province code

// Callback
addressInformationService.getDistrictList("10") { result ->
    when (result) {
        is AddressInformationResult.SuccessDistrict -> {
            // Handle successful district list retrieval
            val districts = result.result
            districts.forEach { district ->
                val districtCode = district.districtCode
                val districtName = district.districtName
                val districtNameEN = district.districtNameEN
                val provinceCode = district.provinceCode
                // Update district dropdown
            }
        }
        is AddressInformationResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### getSubDistrict

Retrieves a list of sub-districts within a specific district and province. Used for populating sub-district dropdowns after district selection.

- Request (caller-supplied)

| Field Name   | Description              | Mandatory | Data Type |
|--------------|--------------------------|-----------|-----------|
| provinceCode | Province code            | M         | String    |
| districtCode | District code to filter  | M         | String    |

- Response (`List<SubDistrict>`)

  HTTP status: 200

> **SubDistrict Fields Reference**
>
> See the complete SubDistrict entity fields table below for all available properties.

- Usage

```kotlin
// Suspend
val result = addressInformationService.getSubDistrict("10", "1001") // Bangkok, Pathumwan

// Callback
addressInformationService.getSubDistrict("10", "1001") { result ->
    when (result) {
        is AddressInformationResult.SuccessSubDistrict -> {
            // Handle successful sub-district list retrieval
            val subDistricts = result.result
            subDistricts.forEach { subDistrict ->
                val subDistrictCode = subDistrict.subDistrictCode
                val subDistrictName = subDistrict.subDistrictName
                val city = subDistrict.city
                val zipCode = subDistrict.zipCode
                // Update sub-district dropdown and auto-fill zip code
            }
        }
        is AddressInformationResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### getCountries

Retrieves a list of all available countries. Used for international address support and country selection.

- Request (caller-supplied)

| Field Name | Description | Mandatory | Data Type |
|------------|-------------|-----------|-----------|
| -          | None        | -         | -         |

- Response (`List<Country>`)

  HTTP status: 200

> **Country Fields Reference**
>
> See the complete Country entity fields table below for all available properties.

- Usage

```kotlin
// Suspend
val result = addressInformationService.getCountries()

// Callback
addressInformationService.getCountries { result ->
    when (result) {
        is AddressInformationResult.SuccessCountries -> {
            // Handle successful countries list retrieval
            val countries = result.result
            countries.forEach { country ->
                val countryId = country.countryId
                val countryCode = country.countryCode
                val countryName = country.countryName
                // Populate country dropdown
            }
        }
        is AddressInformationResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

## Geographic Entity Fields Reference

The AddressInformationUseCase returns several geographic entities for building comprehensive address forms:

### Province Entity Fields

| Field Name       | Description                    | Data Type | Usage Context                    |
|------------------|--------------------------------|-----------|----------------------------------|
| provinceCode     | Unique province identifier     | String?   | Province selection, filtering    |
| provinceName     | Province name in local language| String?   | Display in Thai/local language   |
| provinceNameEN   | Province name in English       | String?   | English interface support        |

**Example Province Data:**
```json
{
  "provinceCode": "10",
  "provinceName": "กรุงเทพมหานคร",
  "provinceNameEN": "Bangkok"
}
```

### District Entity Fields

| Field Name       | Description                    | Data Type | Usage Context                    |
|------------------|--------------------------------|-----------|----------------------------------|
| districtCode     | Unique district identifier     | String?   | District selection, filtering    |
| districtName     | District name in local language| String?   | Display in Thai/local language   |
| districtNameEN   | District name in English       | String?   | English interface support        |
| provinceCode     | Parent province identifier     | String?   | Hierarchy relationship           |

**Example District Data:**
```json
{
  "districtCode": "1001",
  "districtName": "ปทุมวัน",
  "districtNameEN": "Pathumwan",
  "provinceCode": "10"
}
```

### SubDistrict Entity Fields

| Field Name       | Description                    | Data Type | Usage Context                    |
|------------------|--------------------------------|-----------|----------------------------------|
| subDistrictCode  | Unique sub-district identifier | String?   | Sub-district selection, filtering|
| subDistrictName  | Sub-district name              | String?   | Display in forms                 |
| city             | City name                      | String?   | Additional location context      |
| zipCode          | Postal code                    | String?   | Auto-fill zip code fields        |

**Example SubDistrict Data:**
```json
{
  "subDistrictCode": "100101",
  "subDistrictName": "ลุมพินี",
  "city": "กรุงเทพมหานคร",
  "zipCode": "10330"
}
```

### Country Entity Fields

| Field Name       | Description                    | Data Type | Usage Context                    |
|------------------|--------------------------------|-----------|----------------------------------|
| countryId        | Unique country identifier      | String?   | Country selection, reference     |
| countryCode      | ISO country code               | String?   | International standards          |
| countryName      | Country name                   | String?   | Display in forms                 |

**Example Country Data:**
```json
{
  "countryId": "764",
  "countryCode": "TH",
  "countryName": "Thailand"
}
```

---

## Summary

The AddressInformationUseCase provides essential geographic information services for building comprehensive address forms. It supports hierarchical data loading (Province → District → Sub-district) with automatic zip code population, making it ideal for creating user-friendly address input experiences. The UseCase supports both suspend and callback patterns, making it suitable for various architectural approaches while maintaining consistent error handling and data structure throughout the geographic hierarchy.

The service is particularly valuable for applications requiring accurate Thai address data, with built-in support for both local and English language names.