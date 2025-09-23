## AddressUseCase Guide

This guide shows how to initialize and use every public method in `AddressUseCase`, with suspend and callback examples where available. The AddressUseCase provides comprehensive address management functionality including regular addresses, tax addresses, and location information services.

### Getting an instance

```kotlin
val addressService = BuzzebeesSDK.instance().address
```

---

### getAddressList

Retrieves a list of all saved addresses for the current user.

- Request (caller-supplied)

| Field Name | Description | Mandatory | Data Type |
|------------|-------------|-----------|-----------|
| -          | None        | -         | -         |

- Response (`List<Address>`)
  HTTP status: 200

> **Address Fields Reference**
>
> See the complete Address entity fields table below for all available properties.

- Usage

```kotlin
// Suspend
val result = addressService.getAddressList()

// Callback
addressService.getAddressList { result ->
    when (result) {
        is AddressResult.SuccessAddressList -> {
            // Handle successful address list retrieval
            val addresses = result.result
            addresses.forEach { address ->
                val name = address.name
                val addressName = address.addressName
                val fullAddress = address.address
                val isDefault = address.isDefault
            }
        }
        is AddressResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### getAddressDetail

Retrieves detailed information for a specific address using its key.

- Request (caller-supplied)

| Field Name | Description  | Mandatory | Data Type |
|------------|--------------|-----------|-----------|
| key        | Address key  | M         | String    |

- Response (`Address`)
  HTTP status: 200

> **All fields of `Address`** (same as in `getAddressList`)

- Usage

```kotlin
// Suspend
val result = addressService.getAddressDetail("address-key-123")

// Callback
addressService.getAddressDetail("address-key-123") { result ->
    when (result) {
        is AddressResult.SuccessAddressDetail -> {
            // Handle successful address detail retrieval
            val address = result.result
            val fullName = "${address.firstName} ${address.lastName}"
            val contactNumber = address.contactNumber
            val fullAddress = address.address
        }
        is AddressResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### addAddress

Adds a new address to the user's address list.

- Request (caller-supplied)

| Field Name | Description       | Mandatory | Data Type   |
|------------|-------------------|-----------|-------------|
| form       | Address form data | M         | AddressForm |

#### AddressForm Fields

| Field Name        | Description              | Mandatory | Data Type |
|-------------------|--------------------------|-----------|-----------|
| name              | Full name                | O         | String    |
| firstName         | First name               | O         | String    |
| lastName          | Last name                | O         | String    |
| addressName       | Address nickname         | O         | String    |
| contactNumber     | Phone number             | O         | String    |
| taxId             | Tax ID number            | O         | String    |
| address           | Full address text        | O         | String    |
| subDistrictCode   | Sub-district code        | O         | String    |
| subDistrictName   | Sub-district name        | O         | String    |
| districtCode      | District code            | O         | String    |
| districtName      | District name            | O         | String    |
| provinceCode      | Province code            | O         | String    |
| provinceName      | Province name            | O         | String    |
| zipcode           | Postal code              | O         | String    |
| isDefault         | Set as default address   | O         | Boolean   |

- Response (`Address`)
  HTTP status: 200

- Usage

```kotlin
// Create AddressForm for new address
val addressForm = AddressForm(
    key = null, // null for new address
    name = "John Doe",
    firstName = "John",
    lastName = "Doe",
    addressName = "Home",
    contactNumber = "+66812345678",
    taxId = null,
    address = "123 Main Street, Sukhumvit Road",
    subDistrictCode = "100101",
    subDistrictName = "Lumpini",
    districtCode = "1001",
    districtName = "Pathumwan",
    provinceCode = "10",
    provinceName = "Bangkok",
    zipcode = "10330",
    isDefault = true
)

// Suspend
val result = addressService.addAddress(addressForm)

// Callback
addressService.addAddress(addressForm) { result ->
    when (result) {
        is AddressResult.SuccessUpdateAddress -> {
            // Handle successful address creation
            val newAddress = result.result
            val addressName = newAddress.addressName
            val isDefault = newAddress.isDefault
        }
        is AddressResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### updateAddress

Updates an existing address using the provided form data.

- Request (caller-supplied)

| Field Name | Description       | Mandatory | Data Type   |
|------------|-------------------|-----------|-------------|
| form       | Address form data | M         | AddressForm |

#### AddressForm Fields (same as addAddress)

- Response (`Address`) 
  HTTP status: 200

- Usage

```kotlin
// Create AddressForm for updating existing address
val addressForm = AddressForm(
    key = "address-key-123", // existing address key
    name = "John Doe",
    firstName = "John",
    lastName = "Doe",
    addressName = "Home Updated",
    contactNumber = "+66812345678",
    // ... other fields
    isDefault = true
)

// Suspend
val result = addressService.updateAddress(addressForm)

// Callback
addressService.updateAddress(addressForm) { result ->
    when (result) {
        is AddressResult.SuccessUpdateAddress -> {
            // Handle successful address update
            val updatedAddress = result.result
            val addressName = updatedAddress.addressName
            val isDefault = updatedAddress.isDefault
        }
        is AddressResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### deleteAddress

Deletes a specific address using its key.

- Request (caller-supplied)

| Field Name | Description  | Mandatory | Data Type |
|------------|--------------|-----------|-----------|
| key        | Address key  | M         | String    |

- Response (`Address`)  
  HTTP status: 200

- Usage

```kotlin
// Suspend
val result = addressService.deleteAddress("address-key-123")

// Callback
addressService.deleteAddress("address-key-123") { result ->
    when (result) {
        is AddressResult.SuccessDeleteAddress -> {
            // Handle successful address deletion
            val deletedAddress = result.result
            // Address has been successfully deleted
        }
        is AddressResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

## Tax Address Management

The AddressUseCase also provides specialized methods for managing tax addresses, which include additional fields for tax documentation and billing purposes.

---

### getTaxAddressList

Retrieves a list of all saved tax addresses for the current user.

- Request (caller-supplied)

| Field Name | Description | Mandatory | Data Type |
|------------|-------------|-----------|-----------|
| -          | None        | -         | -         |

- Response (`List<Address>`)
  HTTP status: 200

- Usage

```kotlin
// Suspend
val result = addressService.getTaxAddressList()

// Callback
addressService.getTaxAddressList { result ->
    when (result) {
        is AddressResult.SuccessTaxAddressList -> {
            // Handle successful tax address list retrieval
            val taxAddresses = result.result
            taxAddresses.forEach { address ->
                val companyName = address.companyName
                val taxId = address.taxId
                val email = address.email
                val isDefault = address.isDefault
            }
        }
        is AddressResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### getTaxAddressDetail

Retrieves detailed information for a specific tax address using its key.

- Request (caller-supplied)

| Field Name | Description      | Mandatory | Data Type |
|------------|------------------|-----------|-----------|
| key        | Tax address key  | M         | String    |

- Response (`Address`)  
  HTTP status: 200

- Usage

```kotlin
// Suspend
val result = addressService.getTaxAddressDetail("tax-address-key-123")

// Callback
addressService.getTaxAddressDetail("tax-address-key-123") { result ->
    when (result) {
        is AddressResult.SuccessTaxAddressDetail -> {
            // Handle successful tax address detail retrieval
            val taxAddress = result.result
            val companyName = taxAddress.companyName
            val taxId = taxAddress.taxId
            val email = taxAddress.email
            val personType = taxAddress.personType
        }
        is AddressResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### updateTaxAddress

Updates or creates a new tax address using the provided form data. Tax addresses support additional fields for business and tax documentation.

- Request (caller-supplied)

| Field Name | Description           | Mandatory | Data Type   |
|------------|-----------------------|-----------|-------------|
| form       | Tax address form data | M         | AddressForm |

#### Extended AddressForm Fields for Tax Addresses

In addition to the standard address fields, tax addresses support these additional fields:

| Field Name  | Description                      | Mandatory | Data Type |
|-------------|----------------------------------|-----------|-----------|
| email       | Email address                    | O         | String    |
| companyName | Company/Business name            | O         | String    |
| village     | Village name                     | O         | String    |
| building    | Building name                    | O         | String    |
| number      | Building/house number            | O         | String    |
| moo         | Moo (village number)             | O         | String    |
| soi         | Soi (sub-street)                 | O         | String    |
| floor       | Floor number                     | O         | String    |
| road        | Road name                        | O         | String    |
| room        | Room number                      | O         | String    |
| city        | City name                        | O         | String    |
| title       | Title (Mr./Mrs./Ms.)             | O         | String    |
| personType  | Person type (individual/company) | O         | String    |
| countryCode | Country code                     | O         | String    |
| countryName | Country name                     | O         | String    |

- Response (`Address`)  
  HTTP status: 200

- Usage

```kotlin
// Create AddressForm for Tax Address
val taxAddressForm = AddressForm(
    key = "tax-address-key-123", // null for new tax address
    name = "ABC Company Ltd.",
    firstName = "John",
    lastName = "Doe",
    addressName = "Company Headquarters",
    contactNumber = "+6622345678",
    email = "john.doe@abccompany.com",
    companyName = "ABC Company Limited",
    taxId = "0123456789012",
    address = "456 Business District",
    zipcode = "10110",
    subDistrictCode = "101001",
    subDistrictName = "Silom",
    districtCode = "1010",
    districtName = "Bang Rak",
    provinceCode = "10",
    provinceName = "Bangkok",
    countryCode = "TH",
    countryName = "Thailand",
    village = null,
    building = "ABC Tower",
    moo = null,
    soi = "Soi 1",
    floor = "15",
    road = "Silom Road",
    room = "1501",
    city = "Bangkok",
    title = "Mr.",
    personType = "company",
    isDefault = true
)

// Suspend
val result = addressService.updateTaxAddress(taxAddressForm)

// Callback
addressService.updateTaxAddress(taxAddressForm) { result ->
    when (result) {
        is AddressResult.SuccessUpdateTaxAddress -> {
            // Handle successful tax address update/creation
            val updatedTaxAddress = result.result
            val companyName = updatedTaxAddress.companyName
            val taxId = updatedTaxAddress.taxId
            val isDefault = updatedTaxAddress.isDefault
        }
        is AddressResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### deleteTaxAddress

Deletes a specific tax address using its key.

- Request (caller-supplied)

| Field Name | Description      | Mandatory | Data Type |
|------------|------------------|-----------|-----------|
| key        | Tax address key  | M         | String    |

- Response (`Address`)  
  HTTP status: 200

- Usage

```kotlin
// Suspend
val result = addressService.deleteTaxAddress("tax-address-key-123")

// Callback
addressService.deleteTaxAddress("tax-address-key-123") { result ->
    when (result) {
        is AddressResult.SuccessDeleteTaxAddress -> {
            // Handle successful tax address deletion
            val deletedTaxAddress = result.result
            // Tax address has been successfully deleted
        }
        is AddressResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

## Location Information Services

The AddressUseCase also provides geographic information services including provinces, districts, sub-districts, postal codes, and countries data.

---

### getZipCodeList

Retrieves location information by postal/zip code. Returns address details associated with the provided zip code.

- Request (caller-supplied)

| Field Name | Description      | Mandatory | Data Type |
|------------|------------------|-----------|-----------|
| zipCode    | Postal/zip code  | M         | String    |

- Response (`List<Zipcode>`)
  HTTP status: 200

- Usage

```kotlin
// Suspend
val result = addressService.getZipCodeList("10330")

// Callback
addressService.getZipCodeList("10330") { result ->
    when (result) {
        is AddressResult.SuccessZipCodeList -> {
            // Handle successful zip code lookup
            val zipcodes = result.result
            zipcodes.forEach { zipcode ->
                val province = zipcode.province
                val district = zipcode.district
                val subDistrict = zipcode.subDistrict
                val postalCode = zipcode.zipcode
            }
        }
        is AddressResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### getProvinceList

Retrieves a list of all available provinces. Used for address form population and validation.

- Request (caller-supplied)

| Field Name | Description | Mandatory | Data Type |
|------------|-------------|-----------|-----------|
| -          | None        | -         | -         |

- Response (`List<Province>`)
  HTTP status: 200

- Usage

```kotlin
// Suspend
val result = addressService.getProvinceList()

// Callback
addressService.getProvinceList { result ->
    when (result) {
        is AddressResult.SuccessProvinceList -> {
            // Handle successful province list retrieval
            val provinces = result.result
            provinces.forEach { province ->
                val provinceCode = province.provinceCode
                val provinceName = province.provinceName
                val provinceNameEN = province.provinceNameEN
            }
        }
        is AddressResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### getDistrictList

Retrieves a list of districts within a specific province.

- Request (caller-supplied)

| Field Name   | Description              | Mandatory | Data Type |
|--------------|--------------------------|-----------|-----------|
| provinceCode | Province code to filter  | M         | String    |

- Response (`List<District>`)
  HTTP status: 200

- Usage

```kotlin
// Suspend
val result = addressService.getDistrictList("10") // Bangkok province code

// Callback
addressService.getDistrictList("10") { result ->
    when (result) {
        is AddressResult.SuccessDistrictList -> {
            // Handle successful district list retrieval
            val districts = result.result
            districts.forEach { district ->
                val districtCode = district.districtCode
                val districtName = district.districtName
                val districtNameEN = district.districtNameEN
                val provinceCode = district.provinceCode
            }
        }
        is AddressResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### getSubDistrictList

Retrieves a list of sub-districts within a specific district and province.

- Request (caller-supplied)

| Field Name   | Description              | Mandatory | Data Type |
|--------------|--------------------------|-----------|-----------|
| provinceCode | Province code            | M         | String    |
| districtCode | District code to filter  | M         | String    |

- Response (`List<SubDistrict>`)
  HTTP status: 200

- Usage

```kotlin
// Suspend
val result = addressService.getSubDistrictList("10", "1001") // Bangkok, Pathumwan

// Callback
addressService.getSubDistrictList("10", "1001") { result ->
    when (result) {
        is AddressResult.SuccessSubDistrictList -> {
            // Handle successful sub-district list retrieval
            val subDistricts = result.result
            subDistricts.forEach { subDistrict ->
                val subDistrictCode = subDistrict.subDistrictCode
                val subDistrictName = subDistrict.subDistrictName
                val city = subDistrict.city
                val zipCode = subDistrict.zipCode
            }
        }
        is AddressResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### getCountryList

Retrieves a list of all available countries. Used for international address support.

- Request (caller-supplied)

| Field Name | Description | Mandatory | Data Type |
|------------|-------------|-----------|-----------|
| -          | None        | -         | -         |

- Response (`List<Country>`)
  HTTP status: 200

- Usage

```kotlin
// Suspend
val result = addressService.getCountryList()

// Callback
addressService.getCountryList { result ->
    when (result) {
        is AddressResult.SuccessCountryList -> {
            // Handle successful countries list retrieval
            val countries = result.result
            countries.forEach { country ->
                val countryId = country.countryId
                val countryCode = country.countryCode
                val countryName = country.countryName
            }
        }
        is AddressResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

## Address Entity Fields Reference

The Address entity contains comprehensive location and contact information with the following fields:

| Field Name               | Description                      | Data Type | Usage Context    |
|--------------------------|----------------------------------|-----------|------------------|
| **Basic Information**    |                                  |           |                  |
| id                       | Unique identifier                | String?   | All addresses    |
| rowKey                   | Database row key                 | String?   | All addresses    |
| partitionKey             | Database partition key           | String?   | All addresses    |
| name                     | Full name                        | String?   | All addresses    |
| firstName                | First name                       | String?   | All addresses    |
| lastName                 | Last name                        | String?   | All addresses    |
| title                    | Title (Mr./Mrs./Ms.)             | String?   | All addresses    |
| **Contact Information**  |                                  |           |                  |
| contactNumber            | Primary phone number             | String?   | All addresses    |
| homeContactNumber        | Home phone number                | String?   | All addresses    |
| alternateContactNumber   | Alternative phone number         | String?   | All addresses    |
| email                    | Email address                    | String?   | Tax addresses    |
| **Address Details**      |                                  |           |                  |
| addressName              | Address nickname/label           | String?   | All addresses    |
| address                  | Full address text                | String?   | All addresses    |
| house                    | House number                     | String?   | All addresses    |
| number                   | Building/house number            | String?   | All addresses    |
| building                 | Building name                    | String?   | All addresses    |
| village                  | Village name                     | String?   | All addresses    |
| moo                      | Moo (village number)             | String?   | All addresses    |
| soi                      | Soi (sub-street)                 | String?   | All addresses    |
| road                     | Road name                        | String?   | All addresses    |
| street                   | Street name                      | String?   | All addresses    |
| floor                    | Floor number                     | String?   | All addresses    |
| room                     | Room number                      | String?   | All addresses    |
| **Geographic Information** |                                |           |                  |
| subDistrictCode          | Sub-district code                | String?   | All addresses    |
| subDistrictName          | Sub-district name                | String?   | All addresses    |
| districtCode             | District code                    | String?   | All addresses    |
| districtName             | District name                    | String?   | All addresses    |
| provinceCode             | Province code                    | String?   | All addresses    |
| provinceName             | Province name                    | String?   | All addresses    |
| city                     | City name                        | String?   | All addresses    |
| state                    | State name                       | String?   | All addresses    |
| ward                     | Ward name                        | String?   | All addresses    |
| countryCode              | Country code (ISO)               | String?   | All addresses    |
| countryName              | Country name                     | String?   | All addresses    |
| countryId                | Country identifier               | String?   | All addresses    |
| zipcode                  | Postal code                      | String?   | All addresses    |
| **Coordinates**          |                                  |           |                  |
| latitude                 | Latitude coordinate              | Double?   | All addresses    |
| longitude                | Longitude coordinate             | Double?   | All addresses    |
| **Tax Information**      |                                  |           |                  |
| companyName              | Company/Business name            | String?   | Tax addresses    |
| branchName               | Branch name                      | String?   | Tax addresses    |
| taxId                    | Tax identification number        | String?   | Tax addresses    |
| taxName                  | Tax name                         | String?   | Tax addresses    |
| taxNumber                | Tax number                       | String?   | Tax addresses    |
| taxDetail                | Tax details                      | String?   | Tax addresses    |
| **Status Fields**        |                                  |           |                  |
| isDefault                | Default shipping address flag    | Boolean?  | Shipping addresses |
| active                   | Active status                    | Boolean?  | All addresses    |
| personType               | Person type (individual/company) | String?   | Tax addresses    |
| type                     | Address type                     | String?   | All addresses    |
| **System Fields**        |                                  |           |                  |
| createDate               | Creation timestamp               | Long?     | All addresses    |
| modifyDate               | Last modification timestamp      | Long?     | All addresses    |
| timestamp                | Timestamp                        | Int?      | All addresses    |
| eTag                     | Entity tag for caching           | String?   | All addresses    |

---

## Summary

The AddressUseCase provides comprehensive address management functionality for both regular shipping addresses and specialized tax addresses, along with complete location information services. It supports full CRUD operations with both suspend and callback patterns, making it suitable for various architectural approaches. The rich Address entity provides extensive geographic and contact information to support diverse business requirements across different regions and tax jurisdictions.

The UseCase combines three main functional areas: personal address management, tax address management, and location information services (provinces, districts, sub-districts, zip codes, and countries), making it a complete solution for address-related operations in applications requiring Thai geographic data support.