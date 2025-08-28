## AddressUseCase Guide

This guide shows how to initialize and use every public method in `AddressUseCase`, with suspend and callback examples where available. The AddressUseCase provides comprehensive address management functionality including regular addresses and tax addresses.

### Getting an instance

```kotlin
val addressService = BuzzebeesSDK.instance().addressUseCase
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

### updateAddress

Updates or creates a new address using the provided form data.

- Request (caller-supplied)

| Field Name        | Description              | Mandatory | Data Type |
|-------------------|--------------------------|-----------|-----------|
| form              | Address form data        | M         | AddressForm |

#### AddressForm Fields

| Field Name        | Description              | Mandatory | Data Type |
|-------------------|--------------------------|-----------|-----------|
| key               | Address key (for updates) | O       | String    |
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
// Create AddressForm
val addressForm = AddressForm(
    key = "address-key-123", // null for new address
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
    isDefault = true,
    // Other fields...
    email = null,
    companyName = null,
    countryCode = "TH",
    countryName = "Thailand",
    village = null,
    building = null,
    number = "123",
    moo = null,
    soi = null,
    floor = null,
    road = "Sukhumvit Road",
    room = null,
    city = "Bangkok",
    title = "Mr.",
    personType = "individual"
)

// Suspend
val result = addressService.updateAddress(addressForm)

// Callback
addressService.updateAddress(addressForm) { result ->
    when (result) {
        is AddressResult.SuccessUpdateAddress -> {
            // Handle successful address update/creation
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
                val taxAddressName = address.taxAddressName
                val isDefaultTax = address.isDefaultTax
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

| Field Name | Description          | Mandatory | Data Type |
|------------|----------------------|-----------|-----------|
| form       | Tax address form data | M         | AddressForm |

#### Extended AddressForm Fields for Tax Addresses

In addition to the standard address fields, tax addresses support these additional fields:

| Field Name    | Description              | Mandatory | Data Type |
|---------------|--------------------------|-----------|-----------|
| email         | Email address            | O         | String    |
| companyName   | Company/Business name    | O         | String    |
| village       | Village name             | O         | String    |
| building      | Building name            | O         | String    |
| moo           | Moo (village number)     | O         | String    |
| soi           | Soi (sub-street)         | O         | String    |
| floor         | Floor number             | O         | String    |
| road          | Road name                | O         | String    |
| room          | Room number              | O         | String    |
| city          | City name                | O         | String    |
| title         | Title (Mr./Mrs./Ms.)     | O         | String    |
| personType    | Person type (individual/company) | O | String    |
| countryCode   | Country code             | O         | String    |
| countryName   | Country name             | O         | String    |

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
            val isDefaultTax = updatedTaxAddress.isDefaultTax
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
| taxAddressName           | Tax address name                 | String?   | Tax addresses    |
| taxZipcode               | Tax postal code                  | String?   | Tax addresses    |
| taxSubDistrictCode       | Tax sub-district code            | String?   | Tax addresses    |
| taxSubDistrictName       | Tax sub-district name            | String?   | Tax addresses    |
| taxDistrictCode          | Tax district code                | String?   | Tax addresses    |
| taxDistrictName          | Tax district name                | String?   | Tax addresses    |
| taxProvinceCode          | Tax province code                | String?   | Tax addresses    |
| taxProvinceName          | Tax province name                | String?   | Tax addresses    |
| taxSoi                   | Tax soi (sub-street)             | String?   | Tax addresses    |
| **Status Fields**        |                                  |           |                  |
| isDefault                | Default shipping address flag    | Boolean?  | Shipping addresses |
| isDefaultTax             | Default tax address flag         | Boolean?  | Tax addresses    |
| isTax                    | Tax address indicator            | Boolean?  | All addresses    |
| active                   | Active status                    | Boolean?  | All addresses    |
| personType               | Person type (individual/company) | String?   | Tax addresses    |
| type                     | Address type                     | String?   | All addresses    |
| **System Fields**        |                                  |           |                  |
| createDate               | Creation timestamp               | Long?     | All addresses    |
| modifyDate               | Last modification timestamp      | Long?     | All addresses    |
| timestamp                | Timestamp                        | Int?      | All addresses    |
| eTag                     | Entity tag for caching           | String?   | All addresses    |
| **Additional Fields**    |                                  |           |                  |
| idCard                   | ID card number                   | String?   | All addresses    |
| greaterArea              | Greater area name                | String?   | All addresses    |
| landmark                 | Nearby landmark                  | String?   | All addresses    |
| blockNumber              | Block number                     | String?   | All addresses    |
| remark                   | Additional remarks               | String?   | All addresses    |
| extra                    | Extra information                | String?   | All addresses    |
| customInfo1              | Custom information field 1       | String?   | All addresses    |
| customInfo2              | Custom information field 2       | String?   | All addresses    |
| customInfo3              | Custom information field 3       | String?   | All addresses    |
| customInfo4              | Custom information field 4       | String?   | All addresses    |
| customInfo5              | Custom information field 5       | String?   | All addresses    |

---

## Best Practices

### Address Management
- Always validate required fields before calling update methods
- Use meaningful `addressName` values for better user experience
- Set appropriate default addresses for shipping and tax purposes
- Handle network errors gracefully in your UI

### Tax Address Considerations
- Tax addresses require more detailed information for legal compliance
- Always include `companyName` and `taxId` for business addresses
- Use appropriate `personType` values ("individual" or "company")
- Consider local tax regulations when collecting address information

### Error Handling
- Always check for `AddressResult.Error` in callbacks
- Implement retry logic for network failures
- Provide user-friendly error messages
- Validate form data before submission

### Performance Tips
- Cache address lists locally when appropriate
- Use suspend functions for better coroutine integration
- Consider pagination for large address lists
- Minimize API calls by batching operations when possible

---

## Summary

The AddressUseCase provides comprehensive address management functionality for both regular shipping addresses and specialized tax addresses. It supports full CRUD operations with both suspend and callback patterns, making it suitable for various architectural approaches. The rich Address entity provides extensive geographic and contact information to support diverse business requirements across different regions and tax jurisdictions.