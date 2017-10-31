# OfficeEarth API

## Overview

Welcome to the documentation for the OfficeEarth API. This REST API can be used to control all aspects of your account with simple POST and GET requests to our site at https://oe.services. All of the endpoints below should be appended to that base url, for example, to hit the token endpoint you would make a POST request to https://oe.services/api/token. That same structure can be used for all of the endpoints listed below. Some endpoints will need to receive the auth token (which is received from a POST request to /api/token) in the auth header. You will need to add a header with Key/Name Authorization and a value of Bearer { auth_token } where { auth_token } is the token received from the request.

## Endpoints

### POST /api/account - Create new account

Creates a new account with the parameters specified in post request. When logging into the client portal for the first time the user will need to provide a mobile number for verification and then complete the steps given in order to start using the service.

#### Parameters

| **Name**      | **Type**   | **Description**                                           | **Required** |
| ------------- | ---------- | --------------------------------------------------------- | ------------ |
| firstname     | string     | First name of account's primary user (owner)              | Yes          |
| lastname      | string     | Last name of account's primary user                       | Yes          |
| company       | string     | Name of company                                           | Yes          |
| email         | string     | Email address of primary user                             | Yes          |
| password      | string     | Password for primary user                                 | Yes          |
| recaptcha     | string     | Recaptcha response from google for recaptcha verification | Yes          |
| referral_code | string     | Referral code used when signing up                        | No           |

#### Responses

##### Success

HTTP Code: 200

`{ "success": true, "org_id": "123" }`

##### Fail

HTTP Code: 412

`{ "success": false, "errors": { "firstname": [ "firstname is required" ] } }`

`{ "success": false, "error": "Recaptcha is not valid" }`

### POST /api/token - Generate auth token

Send a user's email address and password to receive back a generated authentication token for the user.

#### Parameters

| **Name** | **Type**  | **Description**      | **Required** |
| -------- | --------- | ---------------      | ------------ |
| email    | string    | User's email address | Yes          |
| password | string    | User's password      | Yes          |

#### Responses

##### Success

HTTP Code: 200

`{ "success": true, "token": "hausidahsdf7s#dsafuihsdfaafsd.fdjdf?dasfhaf" }`

##### Fail

HTTP Code: 412

`{ "success": false, "errors": { "email": [ "email is required" ] } }`

HTTP Code: 403

`{ "success": false, "error": "Invalid Credentials" }`

### GET /api/email/{ email } - Check if email is available

Returns a response stating whether or not the email sent in the request is available to be used for creating a new account or not. True means the email is available and false means it is already taken.

#### Responses

##### Success

HTTP Code: 200

`{ "success": true, "available": true }`

##### Fail

HTTP Code: 412

`{ "success": false, "errors": { "email": [ "email is required" ] } }`

### POST /api/forgotpassword - Send password recovery email

Email user associated with sent email an email with a link to recover their password.

#### Parameters

| **Name** | **Type** | **Description** | **Required** |
| -------- | -------- | --------------- | ------------ |
| email    | string   | User's email    | Yes          |

#### Responses

##### Success

HTTP Code: 200

`{ "success": true }`

##### Fail

HTTP Code: 404

`{ "success": false, "error": "User not found" }`

HTTP Code: 412

`{ "success": false, "errors": { "email": [ "email is required" ] } }`

### POST /api/resetpassword - Reset password

Reset user password.

#### Parameters

| **Name** | **Type** | **Description**                                | **Required** |
| -------- | -------- | ---------------------------------------------- | ------------ |
| hash     | string   | Hash sent out to user in forgot password email | Yes          |
| password | string   | New password                                   | Yes          |

#### Responses

##### Success

HTTP Code: 200

`{ "success": true }`

##### Fail

HTTP Code: 412

`{ "success": false, "errors": { "hash": [ "hash is required" ] } }`

HTTP Code: 200

`{ "success": false, "error": "Failed to reset password" }`

### POST /api/account/{ org_id }/user/ - Add new user

Adds a new user to organization specified by the OrganizationUnitID parameter and org_id parameter in the url. A valid email which has not been used for any other OfficeEarth users is required for the setup to be successful. Checking whether the email is taken or not is possible with the email endpoint. 

All passwords have a minimum length of 8 characters long.

The CallConnectNumber must be in international format in order to work and start with a +. An example number for the US would be: +11234567890 and an example number for Australia would be: +61823456789.

AddressTypeID is what sets the user's permissions in the client facing dashboard. The endpoint accepts any one of the values listed below for each user.

| **Value** | **Alias**       | **Permissions granted**                   |
| --------- | --------------- | ----------------------------------------- |
| 5         | Authorized User | Admin with full access                    |
| 6         | User            | Only has access to edit their own details |
| 7         | Manager         | Full access excluding financials          |

Gender signifies a user's gender and the two options are M for a male or F for a female. Other values will not be accepted.

Send_SMS and Phone_Online are both parameters which also only accept a 1 letter value. The value may be Y for allowing SMSes and having their phone online or N for being offline and not receiving SMSes.

If the user wishes to be available all the time for transfers, you may send the value off to Transfer_Off_Time and any hour you'd like to Trasnfer_On_Time and their transfers will be on 24 hours a day. The format of hours accepted by the Transfer_On_Time and Transfer_Off_Time parameters are: 01:00 for 1am or 24:00 for 12am. Any hour value between 01 and 24 and minute value between 00 and 60 may be used.

The Time_Code signifies the internal code used by OfficeEarth for timezones and must be an integer value corresponding to that value. If you would like a timezone not listed here please contact us and we will provide you with the relevant time code.

| **Time Zone**              | **Code** |
| -------------------------- | -------- |
| US Pacific Time / LA       | 6        |
| US Eastern Time / New York | 19       |
| Melbourne                  | 129      |
| Sydney                     | 131      |

#### Parameters

| **Name**           | **Type** | **Description**                                         | **Required** |
| ------------------ | -------- | ------------------------------------------------------- | ------------ |
| OrganizationUnitID | integer  | Id of organization to add user to                       | Yes          |
| FirstName          | string   | User's first name                                       | Yes          |
| LastName           | string   | User's last name                                        | Yes          |
| Email              | string   | User's email                                            | Yes          |
| Password           | string   | User's password                                         | Yes          |
| CallConnectNumber  | string   | User's mobile number                                    | Yes          |
| AddressTypeID      | integer  | User's permission level                                 | Yes          |
| Gender             | string   | User's gender                                           | Yes          |
| Send_SMS           | string   | Whether or not user wishes to receive SMSes             | Yes          |
| Phone_Online       | string   | Whether or not user's phone is online                   | Yes          | 
| Transfer_On_Time   | string   | Time at which which user is available for transfers     | Yes          |
| Transfer_Off_Time  | string   | Time at which user is no longer available for transfers | Yes          |
| Time_Code          | integer  | Code of user's time zone                                | Yes          |

#### Responses

##### Success

HTTP Code: 200

`{ "success": true, "user_id": 12 }`

##### Fail

HTTP Code: 412

`{ "success": false, "errors": { "OrganizationUnitID": [ "OrganizationUnitID is required" ] } }`

### POST /api/account/{ org_id }/user/{ id } - Update user's details

Updates user's details with the values provided presuming they are all in the correct format. If any values are in an illegal format the whole update will fail and nothing will be changed. Any combination of parameters may be sent with the AddressID being required and the other parameters needing to be in the correct format as specified below.

All passwords have a minimum length of 8 characters long.

The CallConnectNumber must be in international format in order to work and start with a +. An example number for the US would be: +11234567890 and an example number for Australia would be: +61823456789.

AddressTypeID is what sets the user's permissions in the client facing dashboard. The endpoint accepts any one of the values listed below for each user.

| **Value** | **Alias**       | **Permissions granted**                   |
| --------- | --------------- | ----------------------------------------- |
| 5         | Authorized User | Admin with full access                    |
| 6         | User            | Only has access to edit their own details |
| 7         | Manager         | Full access excluding financials          |

Gender signifies a user's gender and the two options are M for a male or F for a female. Other values will not be accepted.

Send_SMS and Phone_Online are both parameters which also only accept a 1 letter value. The value may be Y for allowing SMSes and having their phone online or N for being offline and not receiving SMSes.

If the user wishes to be available all the time for transfers, you may send the value off to Transfer_Off_Time and any hour you'd like to Trasnfer_On_Time and their transfers will be on 24 hours a day. The format of hours accepted by the Transfer_On_Time and Transfer_Off_Time parameters are: 01:00 for 1am or 24:00 for 12am. Any hour value between 01 and 24 and minute value between 00 and 60 may be used.

The Time_Code signifies the internal code used by OfficeEarth for timezones and must be an integer value corresponding to that value. If you would like a timezone not listed here please contact us and we will provide you with the relevant time code.

| **Time Zone**              | **Code** |
| -------------------------- | -------- |
| US Pacific Time / LA       | 6        |
| US Eastern Time / New York | 19       |
| Melbourne                  | 129      |
| Sydney                     | 131      |

#### Parameters

| **Name**          | **Type** | **Description**                                         | **Required** |
| ----------------- | -------- | ------------------------------------------------------- | ------------ |
| AddressID         | integer  | User's id                                               | Yes          |
| FirstName         | string   | User's first name                                       | No           |
| LastName          | string   | User's last name                                        | No           |
| Email             | string   | User's email                                            | No           |
| Password          | string   | User's password                                         | No           |
| CallConnectNumber | string   | User's mobile number                                    | No           |
| AddressTypeID     | integer  | User's permission level                                 | No           |
| Gender            | string   | User's gender                                           | No           |
| Send_SMS          | string   | Whether or not user wishes to receive SMSes             | No           |
| Phone_Online      | string   | Whether or not user's phone is online                   | No           | 
| Transfer_On_Time  | string   | Time at which which user is available for transfers     | No           |
| Transfer_Off_Time | string   | Time at which user is no longer available for transfers | No           |
| Time_Code         | integer  | Code of user's time zone                                | No           |

#### Responses

##### Success

HTTP Code: 200

`{ "success": true }`

##### Fail

HTTP Code: 412

`{ "success": false, "errors": { "OrganizationUnitID": [ "AddressID is required" ] } }`

HTTP Code: 404

`{ "success": false, "error": "User not found" }`
