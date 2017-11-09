# OfficeEarth API

## Overview

Welcome to the documentation for the OfficeEarth API. This REST API can be used to control all aspects of your account with simple POST and GET requests to our site at https://oe.services. All of the endpoints below should be appended to that base url, for example, to hit the token endpoint you would make a POST request to https://oe.services/api/token. That same structure can be used for all of the endpoints listed below. Some endpoints will need to receive the auth token (which is received from a POST request to /api/token) in the auth header. You will need to add a header with Key/Name Authorization and a value of Bearer { auth_token } where { auth_token } is the token received from the request.

## Security

All endpoints (unless specified) require a token to be passed in the Authorization header of the request. These endpoints all have the possibility of returning either a 401 status code with an invalid permissions error or a 403 status code with an invalid credentials error. The 401 status code means you do not have permissions to access the resources that were specified. You're either using the wrong auth token or attempting to access a resource with the wrong id. The 403 status code means you're not sending a token. This probably means you forgot to send the token or that it is corrupted. Be sure the header is formatted so that the key is "Authorization" and the value is "Bearer {token}", ensuring a space between the word bearer and the actual token.

#### Unsecured endpoints

- /api/account
- /api/token

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

*Here invalid credentials means that the email address or password are incorrect, as stated above, this route does not need a token to access.

### GET /api/email/{email} - Check if email is available

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

### POST /api/account/{org_id}/user/ - Add new user

*The trailing backslash is not a mistake, if you omit it, then endpoint will not work. 

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

### POST /api/account/{org_id}/user/{id} - Update user's details

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

If the user wishes to be available all the time for transfers, you may send the value off to Transfer_Off_Time and any hour you'd like to Transfer_On_Time and their transfers will be on 24 hours a day. The format of hours accepted by the Transfer_On_Time and Transfer_Off_Time parameters are: 01:00 for 1am or 24:00 for 12am. Any hour value between 01 and 24 and minute value between 00 and 60 may be used.

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

### GET /api/account/{org_id}/users - Get account users

Retrieves list of users belonging to organization defined by org id.

#### Responses

##### Success

HTTP Code: 200

`{ "success": true, "users": [{ "FirstName": "First", "LastName": "Last", "Email": "mail@gmail.com", "Gender": "M", "Phone_Online": "Y", "Send_SMS": "N", "TimeZone": "Australia/Sydney" }] }`

### DELETE /api/account/{org_id}/user/{user_id} - Delete user

Deletes user belonging to organization specified by org id with user id.

#### Responses

##### Success

HTTP Code: 200

`{ "success": true }`

### GET /api/v2/account/{org_id}/call/{call_id}/form_data - Get call form data

Returns form data for the specified call id with all the filled out values.

#### Responses

##### Success

HTTP Code: 200

`{ "success": true, "call_form_data": { "call_id": "1423", "Title": "Custom Form", "Data": { "response": [{ "ID": "43", "Title": "Location", "Value": "None", "API_Feild_ID": "3" }, { "ID": "45", "Title": "Name", "Value": "Jeffrey", "API_Feild_ID": "" }] } } }`

*Feild is not a spelling error, that is unfortunately the field name.

### GET /api/v2/account/{org_id}/calls - Get call history

Get calls made to organization along with full details for the past month.

#### Responses

##### Success

HTTP Code: 200

`{ "success": true, "calls": [{ "id": 1241, "caller_name": "Terry", "caller_phone": "Not given", "duration", "31", "time_date": "2017-05-22 15:34:25" }] }`

## Work In Progress

### GET /api/v2/account/{org_id}/calls/{call_id} - Get call details

Get call details belonging to call specified by the call id.

#### Responses

##### Success

HTTP Code: 200

`{
	"call": {
		"id": "3507",
		"sid": "CA919cfb2e75a8eacafda5f4432caa1589",
		"org_id": "779",
		"time_date": "2017-10-11 09:19:07",
		"duration": "1098",
		"rating": "5",
		"feedback": "Nice one",
		"caller_name": "Ravi",
		"caller_phone": null,
		"caller_message": "He wasn't sure what he wanted so I hung up on him, not really, I just said that someone would call him back.",
		"rule_title": "Looking for Telephone Answering Services",
		"transaction_id": "2846",
		"amount": "15.05",
		"date": "11th Oct / 09:19 am",
		"time": "09:19 am"
	},
	"pass": 1,
	"success": true
}`

### GET /api/v2/account/{org_id}/call/{call_sid}/recording - Get call recording url

Get the url for the recording of the call defined by the call sid.

`{
	"success": true,
	"pass": 1,
	"recordings": [
		"https://api.twilio.com/2010-04-01/Accounts/AC99c96423dfa89b9d38e96f22a6434f78/Calls/CA919cfb2e75a8eacafda5f4432caa1589/Recordings/REa2100d7594515ef25d8c247bfbc701f2"
	]
}`

### GET /api/v2/account/{org_id}/recent_calls - Get recent calls

Get up to last 200 calls for organization specified by org id.

`{
	"success": true,
	"pass": 1,
	"calls": [
		{
			"call_id": "3307",
			"caller_name": "Dani",
			"caller_phone": "0401333800",
			"time_date": "2017-11-08 21:41:45",
			"rule_title": "Looking for Telephone Answering Services",
			"date": "8th Nov / 09:41 pm",
			"time": "09:41 pm"
		},
		{
			"call_id": "399",
			"caller_name": "Dani",
			"caller_phone": "0401333800",
			"time_date": "2017-11-08 21:11:05",
			"rule_title": "Test Calls for Dani",
			"date": "8th Nov / 09:11 pm",
			"time": "09:11 pm"
		},
		{
			"call_id": "365",
			"caller_name": "No name given",
			"caller_phone": "No number given",
			"time_date": "2017-11-08 11:05:33",
			"rule_title": "Telemarketing / Cold Calls",
			"date": "8th Nov / 11:05 am",
			"time": "11:05 am"
		},
		{
			"call_id": "351",
			"caller_name": "Test",
			"caller_phone": " test",
			"time_date": "2017-11-07 16:10:36",
			"rule_title": "Test Calls for Dani",
			"date": "7th Nov / 04:10 pm",
			"time": "04:10 pm"
		}
  ]
},`