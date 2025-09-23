## ProfileUseCase Guide

This guide shows how to initialize and use every public method in `ProfileUseCase`, with suspend
and callback examples where available. The ProfileUseCase provides comprehensive user profile
management functionality for retrieving, updating, and managing user account information.

### Getting an instance

```kotlin
val profileService = BuzzebeesSDK.instance().profileUseCase
```

---

### getProfile

Retrieves the current user's complete profile information.

- Request (caller-supplied)

No parameters required.

- Response (`Profile`)
  HTTP status: 200

### Profile Entity Fields

| Field Name              | Description                     | Data Type         | JSON Field              |
|-------------------------|---------------------------------|-------------------|-------------------------|
| userId                  | User unique identifier          | String?           | UserId                  |
| name                    | Full name                       | String?           | Name                    |
| firstName               | First name                      | String?           | FirstName               |
| lastName                | Last name                       | String?           | LastName                |
| title                   | Title (Mr., Mrs., Ms.)          | String?           | Title                   |
| gender                  | Gender                          | String?           | Gender                  |
| birthDate               | Birth date timestamp            | Long?             | BirthDate               |
| age                     | Age                             | Int?              | Age                     |
| address                 | Address                         | String?           | Address                 |
| subDistrictCode         | Sub-district code               | Int?              | SubDistrictCode         |
| subDistrictName         | Sub-district name               | String?           | SubDistrictName         |
| districtCode            | District code                   | Int?              | DistrictCode            |
| districtName            | District name                   | String?           | DistrictName            |
| provinceCode            | Province code                   | Int?              | ProvinceCode            |
| provinceName            | Province name                   | String?           | ProvinceName            |
| zipcode                 | Postal code                     | String?           | Zipcode                 |
| contactNumber           | Phone number                    | String?           | Contact_Number          |
| email                   | Email address                   | String?           | Email                   |
| notificationEnable      | Notification enabled flag       | Boolean?          | NotificationEnable      |
| postToFacebook          | Facebook posting enabled flag   | Boolean?          | PostToFacebook          |
| locale                  | Language locale                 | Int?              | Locale                  |
| nationalIdCard          | National ID card number         | String?           | NationalIdCard          |
| passport                | Passport number                 | String?           | Passport                |
| membershipId            | Membership ID                   | Int?              | MembershipId            |
| membershipUserName      | Membership username             | String?           | MembershipUserName      |
| loginType               | Login type                      | String?           | LoginType               |
| userLevel               | User level                      | Long?             | UserLevel               |
| displayName             | Display name                    | String?           | DisplayName             |
| playboyCampaignId       | Playboy campaign ID             | String?           | PlayboyCampaignId       |
| userType                | User type                       | String?           | UserType                |
| userTypeName            | User type name                  | String?           | UserTypeName            |
| otherUserId             | Other user ID                   | String?           | OtherUserId             |
| categoryConfig          | Category configuration          | String?           | CategoryConfig          |
| shippingFirstName       | Shipping first name             | String?           | ShippingFirstName       |
| shippingLastName        | Shipping last name              | String?           | ShippingLastName        |
| shippingContactNumber   | Shipping contact number         | String?           | ShippingContactNumber   |
| shippingAddress         | Shipping address                | String?           | ShippingAddress         |
| shippingSubDistrictCode | Shipping sub-district code      | Int?              | ShippingSubDistrictCode |
| shippingSubDistrictName | Shipping sub-district name      | String?           | ShippingSubDistrictName |
| shippingDistrictCode    | Shipping district code          | Int?              | ShippingDistrictCode    |
| shippingDistrictName    | Shipping district name          | String?           | ShippingDistrictName    |
| shippingProvinceCode    | Shipping province code          | Int?              | ShippingProvinceCode    |
| shippingProvinceName    | Shipping province name          | String?           | ShippingProvinceName    |
| shippingZipcode         | Shipping postal code            | String?           | ShippingZipcode         |
| shippingEmail           | Shipping email                  | String?           | ShippingEmail           |
| phonePurchase           | Phone purchase count            | Int?              | PhonePurchase           |
| income                  | Income level                    | String?           | Income                  |
| interests               | Interests                       | String?           | Interests               |
| region                  | Region                          | String?           | Region                  |
| maritalStatus           | Marital status                  | String?           | MaritalStatus           |
| nationality             | Nationality                     | String?           | Nationality             |
| religion                | Religion                        | String?           | Religion                |
| highestEducation        | Highest education level         | String?           | HighestEducation        |
| occupation              | Occupation                      | String?           | Occupation              |
| driverLicenseNumber     | Driver license number           | String?           | DriverLicenseNumber     |
| latitude                | Latitude coordinate             | Double?           | Latitude                |
| longitude               | Longitude coordinate            | Double?           | Longitude               |
| joinDate                | Join date timestamp             | Long?             | JoinDate                |
| village                 | Village                         | String?           | Village                 |
| building                | Building name                   | String?           | Building                |
| number                  | House/building number           | String?           | Number                  |
| moo                     | Village number (Moo)            | String?           | Moo                     |
| room                    | Room number                     | String?           | Room                    |
| floor                   | Floor number                    | String?           | Floor                   |
| soi                     | Soi (lane)                      | String?           | Soi                     |
| city                    | City                            | String?           | City                    |
| road                    | Road                            | String?           | Road                    |
| landmark                | Landmark                        | String?           | Landmark                |
| location                | Location description            | String?           | Location                |
| fullAddress             | Full address                    | String?           | FullAddress             |
| countryCode             | Country code                    | Int?              | CountryCode             |
| countryName             | Country name                    | String?           | CountryName             |
| homeContactNumber       | Home contact number             | String?           | HomeContactNumber       |
| alternateContactNumber  | Alternate contact number        | String?           | AlternateContactNumber  |
| remark                  | Remarks                         | String?           | Remark                  |
| termAndCondition        | Terms and conditions acceptance | Int?              | TermAndCondition        |
| dataPrivacy             | Data privacy acceptance         | Int?              | DataPrivacy             |
| ExtensionJsonProperty   | Extension JSON properties       | Map<String,Any?>? | ExtensionJsonProperty   |
| modifyDate              | Last modified date timestamp    | Long?             | ModifyDate              |
| token                   | Authentication token            | String?           | Token                   |
| updatedPoints           | Updated points information      | UpdatedPoints?    | updated_points          |

- Usage

```kotlin
// Suspend
val result = profileService.getProfile()

// Callback
profileService.getProfile { result ->
    when (result) {
        is ProfileResult.SuccessProfile -> {
            // Handle successful profile retrieval
            val profile = result.result

            val firstName = profile.firstName
            val lastName = profile.lastName
            val email = profile.email
            val contactNumber = profile.contactNumber
            val points = profile.updatedPoints?.points

            println("Name: $firstName $lastName")
            println("Email: $email")
            println("Points: $points")
        }
        is ProfileResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### updateProfile

Updates user profile information with optional image upload.

- Request (caller-supplied)

| Field Name | Description            | Mandatory | Data Type            |
|------------|------------------------|-----------|----------------------|
| file       | Profile image file     | O         | File?                |
| options    | Profile update options | M         | UpdateProfileOptions |

### UpdateProfileOptions Fields

| Field Name             | Description               | Data Type |
|------------------------|---------------------------|-----------|
| firstName              | First name                | String?   |
| lastName               | Last name                 | String?   |
| email                  | Email address             | String?   |
| contactNumber          | Phone number              | String?   |
| notification           | Notification enabled flag | Boolean?  |
| locale                 | Language locale           | Int?      |
| title                  | Title (Mr., Mrs., Ms.)    | String?   |
| gender                 | Gender                    | String?   |
| birthDate              | Birth date timestamp      | Long?     |
| address                | Address                   | String?   |
| subDistrictCode        | Sub-district code         | Int?      |
| subDistrictName        | Sub-district name         | String?   |
| districtCode           | District code             | Int?      |
| districtName           | District name             | String?   |
| provinceCode           | Province code             | Int?      |
| provinceName           | Province name             | String?   |
| countryCode            | Country code              | Int?      |
| countryName            | Country name              | String?   |
| zipCode                | Postal code               | String?   |
| idCard                 | National ID card number   | String?   |
| passport               | Passport number           | String?   |
| maritalStatus          | Marital status            | String?   |
| village                | Village                   | String?   |
| building               | Building name             | String?   |
| number                 | House/building number     | String?   |
| moo                    | Village number (Moo)      | String?   |
| room                   | Room number               | String?   |
| floor                  | Floor number              | String?   |
| soi                    | Soi (lane)                | String?   |
| city                   | City                      | String?   |
| road                   | Road                      | String?   |
| landmark               | Landmark                  | String?   |
| alternateContactNumber | Alternate contact number  | String?   |
| homeContactNumber      | Home contact number       | String?   |
| nationality            | Nationality               | String?   |
| religion               | Religion                  | String?   |
| location               | Location description      | String?   |
| latitude               | Latitude coordinate       | Double?   |
| longitude              | Longitude coordinate      | Double?   |
| income                 | Income level              | String?   |
| interests              | Interests                 | String?   |
| region                 | Region                    | String?   |
| phonePurchase          | Phone purchase count      | Int?      |
| highestEducation       | Highest education level   | String?   |
| occupation             | Occupation                | String?   |
| remark                 | Remarks                   | String?   |
| displayName            | Display name              | String?   |

- Response (`Profile`)
  HTTP status: 200

- Usage

```kotlin
// Create update options
val updateOptions = UpdateProfileOptions().apply {
    firstName = "John"
    lastName = "Doe"
    email = "john.doe@example.com"
    contactNumber = "0812345678"
    gender = "Male"
    locale = 1054
}

// Suspend with image
val result = profileService.updateProfile(
    file = File("/path/to/profile/image.jpg"),
    options = updateOptions
)

// Callback without image
profileService.updateProfile(
    file = null,
    options = updateOptions
) { result ->
    when (result) {
        is ProfileResult.SuccessUpdateProfile -> {
            // Handle successful profile update
            val updatedProfile = result.result
            println("Profile updated successfully")
        }
        is ProfileResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

#### Error Handling

| Error Code | Error ID | Scenario               | User Message                                | Recommended Action                |
|------------|----------|------------------------|---------------------------------------------|-----------------------------------|
| 409        | 1423     | Text too long          | "Text is too long (maximum 100 characters)" | Show character count in real-time |
| 409        | 1904     | Duplicate email/social | "This email is already in use"              | Let user enter new email          |
| 409        | 2118     | National ID duplicate  | "This national ID is already in use"        | Let user check national ID number |
| 400        | -        | Invalid image format   | "Image must be JPG or PNG only"             | Show supported formats            |
| 413        | -        | Image too large        | "Image is too large (maximum 5MB)"          | Suggest reducing image size       |

#### Implementation Example

```kotlin
profileService.updateProfile(file, updateOptions) { result ->
    when (result) {
        is ProfileResult.SuccessUpdateProfile -> {
            showUpdateSuccess()
        }
        is ProfileResult.Error -> {
            val action = when {
                result.error.code == "409" && result.error.id == "1423" ->
                    ErrorAction.ShowTextTooLongError()
                result.error.code == "409" && result.error.id == "1904" ->
                    ErrorAction.ShowDuplicateEmailError()
                result.error.code == "409" && result.error.id == "2118" ->
                    ErrorAction.ShowDuplicateNationalIdError()
                result.error.code == "400" ->
                    ErrorAction.ShowInvalidImageFormatError()
                result.error.code == "413" ->
                    ErrorAction.ShowImageTooLargeError()
                else ->
                    ErrorAction.ShowGenericError(result.error.message)
            }
            handleErrorAction(action)
        }
    }
}
```

---

### updateLanguage

Updates the user's preferred language locale.

- Request (caller-supplied)

| Field Name | Description     | Mandatory | Data Type |
|------------|-----------------|-----------|----------|
| locale     | Language locale | M         | String   |

- Response (`Profile`)
  HTTP status: 200

- Usage

```kotlin
// Suspend
val result = profileService.updateLanguage("1054") // Thai locale

// For English locale
val result = profileService.updateLanguage("1033") // English locale

when (result) {
    is ProfileResult.SuccessUpdateLanguage -> {
        // Handle successful language update
        val profile = result.result
        println("Language updated to locale: ${profile.locale}")
    }
    is ProfileResult.Error -> {
        // Handle error
        val errorCode = result.error.code
        val errorMessage = result.error.message
    }
}

// Callback
profileService.updateLanguage("1054") { result ->
    when (result) {
        is ProfileResult.SuccessUpdateLanguage -> {
            // Handle successful language update
            val profile = result.result
            println("Language updated to locale: ${profile.locale}")
        }
        is ProfileResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

#### Supported Locales

| Locale Code | Language |
|-------------|----------|
| "1033"      | English  |
| "1054"      | Thai     |

---

### updateNotification

Updates user notification preferences.

- Request (caller-supplied)

| Field Name   | Description               | Mandatory | Data Type |
|--------------|---------------------------|-----------|----------|
| notification | Notification enabled flag | M         | Boolean   |

- Response (`Profile`)
  HTTP status: 200

- Usage

```kotlin
// Suspend
val result = profileService.updateNotification(true)

when (result) {
    is ProfileResult.SuccessUpdateNotification -> {
        // Handle successful notification update
        val profile = result.result
        println("Notification enabled: ${profile.notificationEnable}")
    }
    is ProfileResult.Error -> {
        // Handle error
        val errorCode = result.error.code
        val errorMessage = result.error.message
    }
}

// Callback
profileService.updateNotification(true) { result ->
    when (result) {
        is ProfileResult.SuccessUpdateNotification -> {
            // Handle successful notification update
            val profile = result.result
            println("Notification enabled: ${profile.notificationEnable}")
        }
        is ProfileResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### changePassword

Changes the user's password.

- Request (caller-supplied)

| Field Name | Description      | Mandatory | Data Type |
|------------|------------------|-----------|-----------|
| current    | Current password | M         | String    |
| change     | New password     | M         | String    |

- Response (`ChangePasswordResponse`)
  HTTP status: 200

### ChangePasswordResponse Entity Fields

| Field Name | Description   | Data Type | JSON Field |
|------------|---------------|-----------|------------|
| status     | Change status | String?   | status     |

- Usage

```kotlin
// Suspend
val result = profileService.changePassword("currentPassword", "newPassword123")

// Callback
profileService.changePassword("currentPassword", "newPassword123") { result ->
    when (result) {
        is ProfileResult.SuccessChangePassword -> {
            // Handle successful password change
            val response = result.result
            println("Password change status: ${response.status}")
        }
        is ProfileResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

#### Error Handling

| Error Code | Error ID | Scenario                   | User Message                                | Recommended Action                    |
|------------|----------|----------------------------|---------------------------------------------|---------------------------------------|
| 409        | 2090     | Current password incorrect | "Current password is incorrect"             | Let user enter current password again |
| 400        | -        | Password too weak          | "Password must contain numbers and letters" | Show password requirements            |
| 409        | -        | Password recently used     | "Cannot use recent password"                | Let user think of new password        |

#### Implementation Example

```kotlin
profileService.changePassword(currentPassword, newPassword) { result ->
    when (result) {
        is ProfileResult.SuccessChangePassword -> {
            showPasswordChangeSuccess()
        }
        is ProfileResult.Error -> {
            val action = when {
                result.error.code == "409" && result.error.id == "2090" ->
                    ErrorAction.ShowIncorrectCurrentPasswordError()
                result.error.code == "400" ->
                    ErrorAction.ShowWeakPasswordError()
                result.error.code == "409" ->
                    ErrorAction.ShowPasswordRecentlyUsedError()
                else ->
                    ErrorAction.ShowGenericError(result.error.message)
            }
            handleErrorAction(action)
        }
    }
}
```

---

### changeContactNumber

Changes the user's contact number with OTP verification.

- Request (caller-supplied)

| Field Name    | Description             | Mandatory | Data Type |
|---------------|-------------------------|-----------|-----------|
| contactNumber | New contact number      | M         | String    |
| otp           | OTP code                | M         | String    |
| refCode       | Reference code          | M         | String    |
| idCard        | National ID card number | O         | String?   |

- Response (`ChangeContactNumberResponse`)
  HTTP status: 200

### ChangeContactNumberResponse Entity Fields

| Field Name | Description   | Data Type | JSON Field |
|------------|---------------|-----------|------------|
| success    | Success flag  | Boolean?  | success    |
| token      | Updated token | String?   | token      |

- Usage

```kotlin
// Suspend
val result = profileService.changeContactNumber(
    contactNumber = "0898765432",
    otp = "123456",
    refCode = "REF123",
    idCard = "1234567890123"
)

// Callback
profileService.changeContactNumber(
    contactNumber = "0898765432",
    otp = "123456",
    refCode = "REF123",
    idCard = null
) { result ->
    when (result) {
        is ProfileResult.SuccessChangeContactNumber -> {
            // Handle successful contact number change
            val response = result.result
            println("Contact number changed: ${response.success}")
        }
        is ProfileResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

#### Error Handling

| Error Code | Error ID | Scenario                | User Message                    | Recommended Action          |
|------------|----------|-------------------------|---------------------------------|-----------------------------|
| 409        | 2078     | Mobile number duplicate | "This number is already in use" | Let user enter new number   |
| 409        | 2091     | Invalid OTP             | "Invalid OTP code"              | Let user enter OTP again    |
| 400        | -        | Invalid mobile format   | "Invalid phone number format"   | Show correct format example |

#### Implementation Example

```kotlin
profileService.changeContactNumber(contactNumber, otp, refCode, idCard) { result ->
    when (result) {
        is ProfileResult.SuccessChangeContactNumber -> {
            showContactChangeSuccess()
        }
        is ProfileResult.Error -> {
            val action = when {
                result.error.code == "409" && result.error.id == "2078" ->
                    ErrorAction.ShowMobileDuplicateError()
                result.error.code == "409" && result.error.id == "2091" ->
                    ErrorAction.ShowInvalidOtpError()
                result.error.code == "400" ->
                    ErrorAction.ShowInvalidMobileFormatError()
                else ->
                    ErrorAction.ShowGenericError(result.error.message)
            }
            handleErrorAction(action)
        }
    }
}
```

---

### deleteAccount

Deletes the user account.

- Request (caller-supplied)

No parameters required.

- Response (`NoContentResponse`)
  HTTP status: 200

- Usage

```kotlin
// Suspend
val result = profileService.deleteAccount()

// Callback
profileService.deleteAccount { result ->
    when (result) {
        is ProfileResult.SuccessDelete -> {
            // Handle successful account deletion
            println("Account deleted successfully")
        }
        is ProfileResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### getPoint

Retrieves the user's current point balance.

- Request (caller-supplied)

No parameters required.

- Response (`UpdatedPoints`)
  HTTP status: 200

### UpdatedPoints Entity Fields

| Field Name | Description           | Data Type | JSON Field |
|------------|-----------------------|-----------|------------|
| points     | Current point balance | Long?     | points     |
| time       | Last update timestamp | Long?     | time       |

- Usage

```kotlin
// Suspend
val result = profileService.getPoint()

// Callback
profileService.getPoint { result ->
    when (result) {
        is ProfileResult.SuccessGetPoint -> {
            // Handle successful point retrieval
            val points = result.result
            println("Current points: ${points.points}")
        }
        is ProfileResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### getExpirePoint

Retrieves information about points that will expire.

- Request (caller-supplied)

No parameters required.

- Response (`AllExpiringPoints`)
  HTTP status: 200

### AllExpiringPoints Entity Fields

| Field Name     | Description             | Data Type            | JSON Field      |
|----------------|-------------------------|----------------------|-----------------|
| expiringPoints | List of expiring points | `List<ExpiringPoint>?` | expiring_points |

### ExpiringPoint Entity Fields

| Field Name | Description             | Data Type | JSON Field |
|------------|-------------------------|-----------|------------|
| points     | Points that will expire | Int?      | points     |
| time       | Expiration time         | Int?      | time       |

- Usage

```kotlin
// Suspend
val result = profileService.getExpirePoint()

// Callback
profileService.getExpirePoint { result ->
    when (result) {
        is ProfileResult.SuccessGetExpirePoint -> {
            // Handle successful expire point retrieval
            val expirePointsResponse = result.result
            val expiringPoints = expirePointsResponse.expiringPoints

            expiringPoints?.forEach { expirePoint ->
                println("${expirePoint.points} points expire at ${expirePoint.time}")
            }
        }
        is ProfileResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### updateShipping

Updates user shipping information.

- Request (caller-supplied)

| Field Name | Description               | Mandatory | Data Type    |
|------------|---------------------------|-----------|--------------|
| form       | Shipping information form | M         | ShippingForm |

### ShippingForm Fields

| Field Name              | Description                | Data Type |
|-------------------------|----------------------------|-----------|
| shippingFirstName       | Shipping first name        | String?   |
| shippingLastName        | Shipping last name         | String?   |
| shippingProvinceCode    | Shipping province code     | String?   |
| shippingDistrictCode    | Shipping district code     | String?   |
| shippingSubDistrictCode | Shipping sub-district code | String?   |
| shippingZipCode         | Shipping postal code       | String?   |
| shippingAddress         | Shipping address           | String?   |
| shippingProvinceName    | Shipping province name     | String?   |
| shippingDistrictName    | Shipping district name     | String?   |
| shippingSubDistrictName | Shipping sub-district name | String?   |
| shippingContactNumber   | Shipping contact number    | String?   |
| shippingEmail           | Shipping email             | String?   |

- Response (`Shipping`)
  HTTP status: 200

### Shipping Entity Fields

| Field Name              | Description                | Data Type | JSON Field              |
|-------------------------|----------------------------|-----------|-------------------------|
| shippingFirstName       | Shipping first name        | String?   | ShippingFirstName       |
| shippingLastName        | Shipping last name         | String?   | ShippingLastName        |
| shippingContactNumber   | Shipping contact number    | String?   | ShippingContactNumber   |
| shippingAddress         | Shipping address           | String?   | ShippingAddress         |
| shippingSubDistrictCode | Shipping sub-district code | String?   | ShippingSubDistrictCode |
| shippingSubDistrictName | Shipping sub-district name | String?   | ShippingSubDistrictName |
| shippingDistrictCode    | Shipping district code     | String?   | ShippingDistrictCode    |
| shippingDistrictName    | Shipping district name     | String?   | ShippingDistrictName    |
| shippingProvinceCode    | Shipping province code     | String?   | ShippingProvinceCode    |
| shippingProvinceName    | Shipping province name     | String?   | ShippingProvinceName    |
| shippingZipcode         | Shipping postal code       | String?   | ShippingZipcode         |
| shippingEmail           | Shipping email             | String?   | ShippingEmail           |
| rowKey                  | Database row key           | String?   | RowKey                  |
| partitionKey            | Database partition key     | String?   | PartitionKey            |

- Usage

```kotlin
// Create shipping form
val shippingForm = ShippingForm(
    shippingFirstName = "John",
    shippingLastName = "Doe",
    shippingAddress = "123 Main Street",
    shippingProvinceCode = "10",
    shippingDistrictCode = "1001",
    shippingSubDistrictCode = "100101",
    shippingZipCode = "10110",
    shippingProvinceName = "Bangkok",
    shippingDistrictName = "Phra Nakhon",
    shippingSubDistrictName = "Phra Borom Maha Ratchawang",
    shippingContactNumber = "0812345678",
    shippingEmail = "john.doe@example.com"
)

// Suspend
val result = profileService.updateShipping(shippingForm)

// Callback
profileService.updateShipping(shippingForm) { result ->
    when (result) {
        is ProfileResult.SuccessUpdateShipping -> {
            // Handle successful shipping update
            val shipping = result.result
            println("Shipping updated: ${shipping.shippingFirstName} ${shipping.shippingLastName}")
        }
        is ProfileResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### getAddressList

Retrieves the list of user's saved addresses.

- Request (caller-supplied)

No parameters required.

- Response (`List<Address>`)
  HTTP status: 200

### Address Entity Fields

| Field Name              | Description                | Data Type | JSON Field              |
|-------------------------|----------------------------|-----------|-------------------------|
| key                     | Address unique key         | String?   | key                     |
| name                    | Full name                  | String?   | name                    |
| firstName               | First name                 | String?   | firstName               |
| lastName                | Last name                  | String?   | lastName                |
| addressName             | Address nickname           | String?   | addressName             |
| contactNumber           | Contact phone number       | String?   | contactNumber           |
| email                   | Email address              | String?   | email                   |
| companyName             | Company name               | String?   | companyName             |
| taxId                   | Tax ID number              | String?   | taxId                   |
| address                 | Street address             | String?   | address                 |
| zipcode                 | Postal code                | String?   | zipcode                 |
| subDistrictCode         | Sub-district code          | String?   | subDistrictCode         |
| subDistrictName         | Sub-district name          | String?   | subDistrictName         |
| districtCode            | District code              | String?   | districtCode            |
| districtName            | District name              | String?   | districtName            |
| provinceCode            | Province code              | String?   | provinceCode            |
| provinceName            | Province name              | String?   | provinceName            |
| countryCode             | Country code               | String?   | countryCode             |
| countryName             | Country name               | String?   | countryName             |
| village                 | Village                    | String?   | village                 |
| building                | Building name              | String?   | building                |
| number                  | House/building number      | String?   | number                  |
| moo                     | Village number (Moo)       | String?   | moo                     |
| soi                     | Soi (lane)                 | String?   | soi                     |
| floor                   | Floor number               | String?   | floor                   |
| road                    | Road                       | String?   | road                    |
| room                    | Room number                | String?   | room                    |
| city                    | City                       | String?   | city                    |
| title                   | Title (Mr., Mrs., Ms.)     | String?   | title                   |
| personType              | Person type                | String?   | personType              |
| isDefault               | Default address flag       | Boolean?  | isDefault               |

- Usage

```kotlin
// Suspend
val result = profileService.getAddressList()

// Callback
profileService.getAddressList { result ->
    when (result) {
        is ProfileResult.SuccessAddressList -> {
            // Handle successful address list retrieval
            val addresses = result.result
            addresses.forEach { address ->
                println("Address: ${address.addressName} - ${address.address}")
            }
        }
        is ProfileResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### getAddressDetail

Retrieves detailed information for a specific address by key.

- Request (caller-supplied)

| Field Name | Description        | Mandatory | Data Type |
|------------|--------------------|-----------|----------|
| key        | Address unique key | M         | String   |

- Response (`Address`)
  HTTP status: 200

- Usage

```kotlin
// Suspend
val result = profileService.getAddressDetail("address_key_123")

// Callback
profileService.getAddressDetail("address_key_123") { result ->
    when (result) {
        is ProfileResult.SuccessAddressDetail -> {
            // Handle successful address detail retrieval
            val address = result.result
            println("Address: ${address.firstName} ${address.lastName}")
            println("Contact: ${address.contactNumber}")
            println("Full Address: ${address.address}, ${address.districtName}, ${address.provinceName}")
        }
        is ProfileResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### updateAddress

Creates a new address or updates an existing address.

- Request (caller-supplied)

| Field Name | Description  | Mandatory | Data Type   |
|------------|--------------|-----------|-------------|
| form       | Address form | M         | AddressForm |

### AddressForm Fields

| Field Name              | Description                | Data Type |
|-------------------------|----------------------------|-----------|
| key                     | Address key (null for new)| String?   |
| name                    | Full name                  | String?   |
| firstName               | First name                 | String?   |
| lastName                | Last name                  | String?   |
| addressName             | Address nickname           | String?   |
| contactNumber           | Contact phone number       | String?   |
| email                   | Email address              | String?   |
| companyName             | Company name               | String?   |
| taxId                   | Tax ID number              | String?   |
| address                 | Street address             | String?   |
| zipcode                 | Postal code                | String?   |
| subDistrictCode         | Sub-district code          | String?   |
| subDistrictName         | Sub-district name          | String?   |
| districtCode            | District code              | String?   |
| districtName            | District name              | String?   |
| provinceCode            | Province code              | String?   |
| provinceName            | Province name              | String?   |
| countryCode             | Country code               | String?   |
| countryName             | Country name               | String?   |
| village                 | Village                    | String?   |
| building                | Building name              | String?   |
| number                  | House/building number      | String?   |
| moo                     | Village number (Moo)       | String?   |
| soi                     | Soi (lane)                 | String?   |
| floor                   | Floor number               | String?   |
| road                    | Road                       | String?   |
| room                    | Room number                | String?   |
| city                    | City                       | String?   |
| title                   | Title (Mr., Mrs., Ms.)     | String?   |
| personType              | Person type                | String?   |
| isDefault               | Default address flag       | Boolean?  |

- Response (`Address`)
  HTTP status: 200

- Usage

```kotlin
// Create address form
val addressForm = AddressForm(
    key = null, // null for new address, provide key for update
    name = null,
    firstName = "John",
    lastName = "Doe",
    addressName = "Home",
    contactNumber = "0812345678",
    email = "john.doe@example.com",
    companyName = null,
    taxId = null,
    address = "123 Main Street",
    zipcode = "10110",
    subDistrictCode = "100101",
    subDistrictName = "Phra Borom Maha Ratchawang",
    districtCode = "1001",
    districtName = "Phra Nakhon",
    provinceCode = "10",
    provinceName = "Bangkok",
    countryCode = "764",
    countryName = "Thailand",
    village = null,
    building = null,
    number = "123",
    moo = null,
    soi = null,
    floor = null,
    road = "Main Street",
    room = null,
    city = "Bangkok",
    title = "Mr.",
    personType = "individual",
    isDefault = true
)

// Suspend
val result = profileService.updateAddress(addressForm)

// Callback
profileService.updateAddress(addressForm) { result ->
    when (result) {
        is ProfileResult.SuccessUpdateAddress -> {
            // Handle successful address update
            val address = result.result
            println("Address updated: ${address.addressName}")
        }
        is ProfileResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### deleteAddress

Deletes a user address by key.

- Request (caller-supplied)

| Field Name | Description        | Mandatory | Data Type |
|------------|--------------------|-----------|----------|
| key        | Address unique key | M         | String   |

- Response (`NoContentResponse`)
  HTTP status: 200

- Usage

```kotlin
// Suspend
val result = profileService.deleteAddress("address_key_123")

// Callback
profileService.deleteAddress("address_key_123") { result ->
    when (result) {
        is ProfileResult.SuccessDeleteAddress -> {
            // Handle successful address deletion
            println("Address deleted successfully")
        }
        is ProfileResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### getTaxAddressList

Retrieves the list of user's saved tax addresses.

- Request (caller-supplied)

No parameters required.

- Response (`List<Address>`)
  HTTP status: 200

- Usage

```kotlin
// Suspend
val result = profileService.getTaxAddressList()

// Callback
profileService.getTaxAddressList { result ->
    when (result) {
        is ProfileResult.SuccessTaxAddressList -> {
            // Handle successful tax address list retrieval
            val taxAddresses = result.result
            taxAddresses.forEach { address ->
                println("Tax Address: ${address.companyName} - ${address.taxId}")
            }
        }
        is ProfileResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### getTaxAddressDetail

Retrieves detailed information for a specific tax address by key.

- Request (caller-supplied)

| Field Name | Description            | Mandatory | Data Type |
|------------|------------------------|-----------|----------|
| key        | Tax address unique key | M         | String   |

- Response (`Address`)
  HTTP status: 200

- Usage

```kotlin
// Suspend
val result = profileService.getTaxAddressDetail("tax_address_key_456")

// Callback
profileService.getTaxAddressDetail("tax_address_key_456") { result ->
    when (result) {
        is ProfileResult.SuccessTaxAddressDetail -> {
            // Handle successful tax address detail retrieval
            val taxAddress = result.result
            println("Company: ${taxAddress.companyName}")
            println("Tax ID: ${taxAddress.taxId}")
            println("Address: ${taxAddress.address}")
        }
        is ProfileResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### updateTaxAddress

Creates a new tax address or updates an existing tax address.

- Request (caller-supplied)

| Field Name | Description      | Mandatory | Data Type   |
|------------|------------------|-----------|-------------|
| form       | Tax address form | M         | AddressForm |

- Response (`Address`)
  HTTP status: 200

- Usage

```kotlin
// Create tax address form
val taxAddressForm = AddressForm(
    key = null, // null for new tax address, provide key for update
    name = null,
    firstName = "John",
    lastName = "Doe",
    addressName = "Company Office",
    contactNumber = "0212345678",
    email = "accounting@company.com",
    companyName = "ABC Company Ltd.",
    taxId = "0123456789012",
    address = "456 Business Street",
    zipcode = "10110",
    subDistrictCode = "100102",
    subDistrictName = "Wang Burapha Phirom",
    districtCode = "1001",
    districtName = "Phra Nakhon",
    provinceCode = "10",
    provinceName = "Bangkok",
    countryCode = "764",
    countryName = "Thailand",
    village = null,
    building = "ABC Tower",
    number = "456",
    moo = null,
    soi = null,
    floor = "15",
    road = "Business Street",
    room = "1501",
    city = "Bangkok",
    title = "Ms.",
    personType = "company",
    isDefault = true
)

// Suspend
val result = profileService.updateTaxAddress(taxAddressForm)

// Callback
profileService.updateTaxAddress(taxAddressForm) { result ->
    when (result) {
        is ProfileResult.SuccessUpdateTaxAddress -> {
            // Handle successful tax address update
            val taxAddress = result.result
            println("Tax address updated: ${taxAddress.companyName}")
        }
        is ProfileResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### deleteTaxAddress

Deletes a user tax address by key.

- Request (caller-supplied)

| Field Name | Description            | Mandatory | Data Type |
|------------|------------------------|-----------|----------|
| key        | Tax address unique key | M         | String   |

- Response (`NoContentResponse`)
  HTTP status: 200

- Usage

```kotlin
// Suspend
val result = profileService.deleteTaxAddress("tax_address_key_456")

// Callback
profileService.deleteTaxAddress("tax_address_key_456") { result ->
    when (result) {
        is ProfileResult.SuccessDeleteTaxAddress -> {
            // Handle successful tax address deletion
            println("Tax address deleted successfully")
        }
        is ProfileResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

## Summary

The ProfileUseCase provides comprehensive user profile management functionality within the Buzzebees
SDK. It offers methods to retrieve and update user profiles, manage language and notification settings,
manage shipping information, change passwords and contact numbers, track point balances and expiring points,
manage user addresses (both regular and tax addresses), and delete user accounts. The UseCase supports both
basic profile operations and advanced features like image upload, OTP verification for sensitive operations,
and comprehensive address management for delivery and tax purposes.

The ProfileUseCase includes **17 methods** covering:

**Core Profile Operations (7 methods)**
- `getProfile()` - Retrieve complete user profile
- `updateProfile()` - Update profile with optional image upload
- `updateLanguage()` - Update user language preference
- `updateNotification()` - Update notification settings
- `changePassword()` - Change user password
- `changeContactNumber()` - Change contact number with OTP verification
- `deleteAccount()` - Delete user account

**Points Management (2 methods)**
- `getPoint()` - Get current point balance
- `getExpirePoint()` - Get expiring points information

**Shipping Operations (1 method)**
- `updateShipping()` - Update shipping information

**Address Management (8 methods)**
- `getAddressList()` - Get list of saved addresses
- `getAddressDetail()` - Get specific address details
- `updateAddress()` - Create or update address
- `deleteAddress()` - Delete address
- `getTaxAddressList()` - Get list of saved tax addresses
- `getTaxAddressDetail()` - Get specific tax address details
- `updateTaxAddress()` - Create or update tax address
- `deleteTaxAddress()` - Delete tax address

All methods support both suspend functions and callback patterns for flexible integration with different
architectural approaches and coroutine usage preferences.
