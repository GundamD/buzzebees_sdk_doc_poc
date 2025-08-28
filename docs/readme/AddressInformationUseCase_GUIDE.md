## AddressInformationUseCase Guide

This guide shows how to initialize and use every public method in `AddressInformationUseCase`, with suspend and callback examples where available. The AddressInformationUseCase provides geographic location data for building address hierarchies, particularly for Thai administrative divisions.

### Getting an instance

```kotlin
val addressInfoService = BuzzebeesSDK.instance().addressInformationUseCase
```

---

### getProvinceList

Retrieves a list of all provinces (top-level administrative divisions).

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
val result = addressInfoService.getProvinceList()

// Callback
addressInfoService.getProvinceList { result ->
    when (result) {
        is AddressInformationResult.SuccessProvince -> {
            // Handle successful province list retrieval
            val provinces = result.result
            provinces.forEach { province ->
                val code = province.provinceCode
                val nameTH = province.provinceName
                val nameEN = province.provinceNameEN
                println("Province: $nameTH ($nameEN) - Code: $code")
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

Retrieves a list of districts (second-level administrative divisions) within a specific province.

- Request (caller-supplied)

| Field Name   | Description   | Mandatory | Data Type |
|--------------|---------------|-----------|-----------|
| provinceCode | Province code | M         | String    |

- Response (`List<District>`) 
  HTTP status: 200

> **District Fields Reference**
>
> See the complete District entity fields table below for all available properties.

- Usage

```kotlin
// Suspend
val result = addressInfoService.getDistrictList("10") // Bangkok province code

// Callback
addressInfoService.getDistrictList("10") { result ->
    when (result) {
        is AddressInformationResult.SuccessDistrict -> {
            // Handle successful district list retrieval
            val districts = result.result
            districts.forEach { district ->
                val code = district.districtCode
                val nameTH = district.districtName
                val nameEN = district.districtNameEN
                val provinceCode = district.provinceCode
                println("District: $nameTH ($nameEN) - Code: $code, Province: $provinceCode")
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

Retrieves a list of sub-districts (third-level administrative divisions) within a specific district and province.

- Request (caller-supplied)

| Field Name   | Description   | Mandatory | Data Type |
|--------------|---------------|-----------|-----------|
| provinceCode | Province code | M         | String    |
| districtCode | District code | M         | String    |

- Response (`List<SubDistrict>`) 
  HTTP status: 200

> **SubDistrict Fields Reference**
>
> See the complete SubDistrict entity fields table below for all available properties.

- Usage

```kotlin
// Suspend
val result = addressInfoService.getSubDistrict("10", "1001") // Bangkok, Pathumwan

// Callback
addressInfoService.getSubDistrict("10", "1001") { result ->
    when (result) {
        is AddressInformationResult.SuccessSubDistrict -> {
            // Handle successful sub-district list retrieval
            val subDistricts = result.result
            subDistricts.forEach { subDistrict ->
                val code = subDistrict.subDistrictCode
                val name = subDistrict.subDistrictName
                val city = subDistrict.city
                val zipCode = subDistrict.zipCode
                println("SubDistrict: $name - Code: $code, City: $city, Zip: $zipCode")
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

## Entity Fields Reference

### Province Entity

The Province entity represents top-level administrative divisions with the following fields:

| Field Name     | Description                    | Data Type | JSON Field        |
|----------------|--------------------------------|-----------|-------------------|
| provinceCode   | Unique province identifier     | String?   | province_code     |
| provinceName   | Province name in Thai          | String?   | province_name     |
| provinceNameEN | Province name in English       | String?   | province_name_eng |

**Example Province Data:**
```json
{
  "province_code": "10",
  "province_name": "กรุงเทพมหานคร",
  "province_name_eng": "Bangkok"
}
```

### District Entity

The District entity represents second-level administrative divisions with the following fields:

| Field Name     | Description                    | Data Type | JSON Field       |
|----------------|--------------------------------|-----------|------------------|
| districtCode   | Unique district identifier     | String?   | district_code    |
| districtName   | District name in Thai          | String?   | district_name    |
| districtNameEN | District name in English       | String?   | district_name_EN |
| provinceCode   | Parent province identifier     | String?   | province_code    |

**Example District Data:**
```json
{
  "district_code": "1001",
  "district_name": "ปทุมวัน",
  "district_name_EN": "Pathumwan",
  "province_code": "10"
}
```

### SubDistrict Entity

The SubDistrict entity represents third-level administrative divisions with the following fields:

| Field Name        | Description                    | Data Type | JSON Field        |
|-------------------|--------------------------------|-----------|-------------------|
| subDistrictCode   | Unique sub-district identifier | String?   | subdistrict_code  |
| subDistrictName   | Sub-district name in Thai      | String?   | subdistrict_name  |
| city              | City name                      | String?   | city              |
| zipCode           | Postal code                    | String?   | zip_code          |

**Example SubDistrict Data:**
```json
{
  "subdistrict_code": "100101",
  "subdistrict_name": "ลุมพินี",
  "city": "กรุงเทพมหานคร",
  "zip_code": "10330"
}
```

## Summary

The AddressInformationUseCase provides essential geographic data services for building robust address management systems. It offers a hierarchical approach to Thai administrative divisions with bilingual support (Thai/English), making it ideal for applications serving both local and international users. The service integrates seamlessly with the AddressUseCase to provide complete address lifecycle management with proper validation and standardization.