# Vincere Identity #

Vincere Identity give partners the ability to access Vincere Rest API since v2. Using Oauth 2.0 as the authorization mechanism for the Vincere Rest API, developers now can obtain OAuth keys for developing applications with Vincere REST API.

To start with Vincere Oauth2, you must have a valid OAuth 2.0 data below: 

parameters | description
-----------| -----------
redirect_uri | URI for the location of your application provided to Vincere when registered
client_id | ID generated by Vincere to identity your account

All of those can be obtained by going to **Vincere application -> Settings -> App Store and look for the API Authentication & Throttling tab**. 

There are 2 environments provided for Vincere Identity: production at **https://id.vincere.io**, and test at: **https://id.vinceredev.com**. Please contact us in case you need a test one.  

## Complete the authorization flow ##

* ### Get an authorization code ###

>**GET** /oauth2/authorize

The /oauth2/authorize endpoint only supports HTTPS GET. The user pool client typically makes this request through the system browser. This step would return you access_token + id_token (expired in 1 hour) and refresh_token to refresh the 2 ones. 

**Request parameters**

parameters | location | description 
-----------| ---------| -----------
client_id | query param | **Required**. your client_id generated by Vincere
state | query param | **Optional** The param would be returned with authorization_code
redirect_uri | query param | **Required** your redirect uri that registered with Vincere. 
response_type | query param | **Required** Only accept *code* 

Sample Request

> **GET** https://id.vincere.io/oauth2/authorize?client_id=123456&state=STATE&redirect_uri=https://YOUR_APP/redirect_uri&response_type=code

**Response**

Vincere Identity redirects back to your app with authorization code and your state. The code and state must be returned in the query string parameters.

Sample Response 

> https://YOUR_APP/redirect_uri?code=AUTHORIZATION_CODE&state=STATE

*It is highly recommended that you have to store refresh_token securedly (for example server-side to avoid exposing it externally).* 

* ### Get tokens ###

>**POST** /oauth2/token  
>Content-Type: application/x-www-form-urlencoded

The /oauth2/token endpoint only supports HTTPS POST. Right after receiving authorization code, your responsibility is to make first request to this endpoint to get refresh_token, access_token and id_token. **Remember that refresh_token is allocated only one time/code. Repeated calls would result in *invalid_grant*** Following calls with refresh token in body would return only new id_token + access_token. 
 
**Request Parameters**

parameters | location | description 
-----------| ---------| -----------
client_id | query param | **Required**. your client_id generated by Vincere
code | body | **Optional** Authorization code. Required in case grant_type is *authorization_code*
grant_type | body | **Required** *authorization_code* OR *refresh_token*
refresh_token | body | **Optional** refresh token. Required in case grant_type is *refresh_token* 

**Response**

Depending on your requests to token endpoint, the response would return refresh_token + access_token + id_token (in case of grant_type=authorization_code), or only access_token + id_token in case of grant_type=refresh_token. 

Sample Response

```json
Content-Type: application/json

{ 
 "access_token":"634563675783dfjh", 
 "refresh_token":"hgsgf6343g5gg", 
 "id_token":"7765634hhg4fggf",
 "token_type":"Bearer", 
 "expires_in":3600
}
```

**Error**

*invalid_request*

The request is missing a required parameter, includes an unsupported parameter value, or is otherwise malformed. For example, grant_type is refresh_token but refresh_token is not included.

*invalid_client*

Client authentication failed. 

*invalid_grant*

Refresh token has been revoked.

Authorization code has been consumed already or does not exist.

* ### LOGOUT Endpoint ###

>**GET** /oauth2/logout

The /oauth2/logout endpoint only supports HTTPS GET. Logout action would clear out the existing session and shows the login screen, using the same parameters for GET /oauth2/authorize.


**Request parameters**

parameters | location | description 
-----------| ---------| -----------
client_id | query param | **Required**. your client_id generated by Vincere
state | query param | **Optional** The param would be returned with authorization_code
redirect_uri | query param | **Required** your redirect uri that registered with Vincere. 
response_type | query param | **Required** Only accept *code* 


* ### Get user services ###

>**GET** /oauth2/user  
>Header: token: id_token

To get the list of Vincere service that authenticated user is using, you need to call this API. 

**Request parameters**

parameters | location | description 
-----------| ---------| -----------
id_token | header param | **Required**. user's id_token

**Response**

List of services related to user would be returned in JSON format. 

```json
Content-Type: application/json

{ 
 "email":"email@vincere.io", 
 "tenants": [
 	{
 		"tenant": "abc.vincere.io",
 		"userId": 1234
 	},
 	{
 		"tenant": "abc1.vincere.io",
 		"userId": 123
 	}
 ]
}
```
* ### Token to use APIs v2 ###

After obtaining tokens you can make REST calls that get data or perform CRUD operations on Vincere data. The request has to pass in token: id_token as header param. Example is below: 

> https://<YOUR_VINCERE_DOMAIN>/api/v2/candidate/1234  
> Header: id_token: id_token

For more information about OAuth2, please refer to https://www.digitalocean.com/community/tutorials/an-introduction-to-oauth-2 
