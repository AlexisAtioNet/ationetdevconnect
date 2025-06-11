![ationetlogo](Content/Images/ATIOnetLogo_250x70.png)
# Ationet Fleet Mobile Payment - Site System Implementation #

|Document Information||
|--- |--- |
|Archivo:|ATIONet - Fleet Mobile Payment - Site System Implementation|
|Doc Version:|1.0|
|Date:|11-06-2025|
|Author:|Alexis E. Yañez|

|Change Control |||
|--- |--- |--- |
|Ver.|Date|Changes|
|1.0|11-06-2025|Initial version.|

## Content ##
- [Introduction](#introduction)
- [Authentication](#authentication)
- [Configuration Messaging](#configuration-messaging)
  - [Initialization Flow](#initialization-flow)
  - [Connection](#Connection)
  - [Mobile Events](#Mobile_Events)
  - [Site Data](#site-data)
  - [Products](#products)
  - [Dispensers](#Dispensers)
  - [Modes](#Modes)
  - [Country Settings](#country-settings)


## Introduction
This document describes the operation to make the connection between the Site System and Ationet in order to operate.

## Authentication

Steps to Configure Basic Authentication
Credential Encryption:

Combines the username and password into a single string, separated by a colon (:).
Encode this string in Base64.
Example:

username:password

Encoded in Base64:

bm9tYnJlX2RlX3VzdWFyaW86Y29udHJhc2XDsWE=


Add the Authorization Header:

Include the Base64 encoded string in the HTTP request header.
The format of the header is:
Authorization: Basic <credenciales_codificadas>
Example:

Authorization: Basic bm9tYnJlX2RlX3VzdWFyaW86Y29udHJhc2XDsWE=
Make HTTP Request:

Sends the HTTP request with the authorization header included.
The server will decode the credentials and verify them.
HTTP Request Example

GET /recurso HTTP/1.1
Host: api.ejemplo.com
Authorization: Basic bm9tYnJlX2RlX3VzdWFyaW86Y29udHJhc2XDsWE=



## Configuration Messaging

### Initialization Flow

<div align="center">
  <img src="https://github.com/user-attachments/assets/1a3643c2-3583-4efc-afa3-407f0b6ebfeb" alt="Diagrama de Secuencia de Conexión SSE">
</div>


### Connection

<b>Relative URL:</b> /connection </br>
<b>Method:</b> POST </br>
<b>Input:</b> application/json </br>
<b>Output:</b> application/json </br>
<b>Use:</b> Connection verification and heartbeat. The Site System will send this message periodically every 45 seconds. If no response is received, the SSE connection will be closed, which will be re-established as soon as the MPPA successfully responds to this message.
If a response is received and the Status Code is 400 (Bad Request), the Site System will close the SSE connection (if open) and a new connection will be established.

<b>Request Body:</b>
```json
{
  "applicationSender": "{{SiteCode}}",
  "workstationID": "{{SiteCode}}",
  "timestamp": "2009-11-20T17:30:50",
  "interfaceVersion": "1.0"
}
```

<b>Definition of Fields:</b> </br>
<b>applicationSender:</b> It must contain the SiteCode of the site you are operating.</br>
<b>workstationID:</b>   Must contain the SiteCode of the site you are operating.</br>
<b>timestamp:</b> date and time of request</br>
<b>interfaceVersion:</b> fixed</br>

<b>Response Body:</b>
You get an HTTP Response 200 or 400 depending on whether the request was successfully processed, without a body.


### Mobile Events

<b>Relative URL:</b> /mobileEvents </br>
<b>Method:</b> GET </br>
<b>Input:</b> application/json </br>
<b>Output:</b> application/json </br>
<b>Use:</b> It allows requesting a new session to the MPPA Host in order to be used to operate. In its response it delivers a URL (url property of the body of the response) with which the following configuration messages will be executed and the session ID with which it will operate can be obtained.
The session configuration messages are made by adding the action to be configured to this URL.

<b>Request Header:</b> Inside the header you must send the Key ApplicationSender whose value will be the SiteCode used. This is done to perform an early detection of which site is requesting the creation of a new session.

<b>Request Body:</b>
Empty

<b>Response Body:</b>
You get an HTTP Response 200 or 400 depending on whether the request could be processed correctly. 
The Body of the response is the following, where the errorCode property must contain the value “ERRCD_OK” which indicates that the configuration was successful.

```json
{
    "url": "URLAmbiente/v1/SiteSystem/sse/c14e1013-5acd-43d5-aad4-68679f295a39",
    "errorCode": "ERRCD_OK",
    "endpointType": "SSE"
}
```


### Site Data

<b>Relative URL:</b> URLAmbiente/v{{Version}}/SiteSystem/SSE/{{SiteSessionId}}/siteData </br>
<b>Method:</b> POST </br>
<b>Configuration Type:</b> Mandatory </br>
<b>Input:</b> application/json </br>
<b>Output:</b> application/json </br>
<b>Use:</b> It allows configuring in the session created in the MobileEvents, the site which is operating. This is done in the Id property of the siteIDs object.

<b>Request Body:</b>
```json
{
     "name" : "IFSF/Conexxus Station",   
     "siteIDs" : [
             { "type":"SHIPTO", "id": "{{SiteCode}}" } 
     ],
     "addressLines" : [
       "Delta 1A, Building L’Aimant",   
       "Business Park Ijsseloord 2"
     ]
}
```

<b>Response Body:</b>
You get an HTTP Response 200 or 400 depending on whether the request could be processed correctly. 
The Body of the response is the following, where the error property must contain the value “ERRCD_OK” which indicates that the configuration was successful.

```json
{
    "timestamp": "2025-03-21T21:33:23.4991943+00:00",
    "result": "success",
    "error": "ERRCD_OK",
    "message": "Operation completed successfully"
}
```



### Products

<b>Relative URL:</b> URLAmbiente/v{{Version}}/SiteSystem/SSE/{{SiteSessionId}}/products </br>
<b>Method:</b> POST </br>
<b>Configuration Type:</b> Obligatoria </br>
<b>Input:</b> application/json </br>
<b>Output:</b> application/json </br>
<b>Use:</b> It allows configuring in the session created in the MobileEvents, the products with which the site will operate. This is done with the fuelProducts object. </br> </br>
<b>IMPORTANT: the fuelPrice property inside the product object that travels in the “fuelProducts” array is mandatory since the fleet authorizer takes the value of this property to operate..</b>

<b>Request Body:</b>
```json
{
   "fuelProducts":[
      {
         "productNo":"1",
         "productName":"Premium",
         "productCode":"CODE1",
         "prices":[
            {
               "fuelUnitPrice":{
                  "value":"20.00",
                  "currency":""
               },
               "priceTier":"cash",
               "modeNo":"1"
            },
            {
               "fuelUnitPrice":{
                  "value":"25.00",
                  "currency":""
               },
               "priceTier":"cash",
               "modeNo":"2"
            }
         ],
         "fuelPrice": {
            "value":"20.00",
            "currency":"USD"
         }
      },
      {
         "productNo":"2",
         "productName":"Diesel",
         "productCode":"CODE2",
         "prices":[
            {
               "fuelUnitPrice":{
                  "value":"30.00",
                  "currency":""
               },
               "priceTier":"cash",
               "modeNo":"1"
            },
            {
               "fuelUnitPrice":{
                  "value":"35.00",
                  "currency":""
               },
               "priceTier":"cash",
               "modeNo":"2"
            }
         ],
         "fuelPrice": {
            "value":"10.00",
            "currency":"USD"
         }
      }
   ]
}
```

<b>Response Body:</b>
You get an HTTP Response 200 or 400 depending on whether the request could be processed correctly. 
The Body of the response is the following, where the error property must contain the value "ERRCD_OK" which indicates that the configuration was successful.

```json
{
    "timestamp": "2025-03-21T21:33:23.4991943+00:00",
    "result": "success",
    "error": "ERRCD_OK",
    "message": "Operation completed successfully"
}
```


### Dispensers

<b>Relative URL:</b> URLAmbiente/v{{Version}}/SiteSystem/SSE/{{SiteSessionId}}/dsps </br>
<b>Method:</b> POST </br>
<b>Configuration Type:</b> Obligatoria </br>
<b>Input:</b> application/json </br>
<b>Output:</b> application/json </br>
<b>Use:</b> Allows to configure in the session created in the MobileEvents, the list of Dispensers with which the site will operate. This is done with the dispensersConfiguration object.

<b>Request Body:</b>
```json
{
  "dispensersConfiguration": [
    {
      "dispenserID": "1",
      "fuelingPoints": [
        {
          "fuelingPointID": "1",
          "nozzles": [
            {
              "nozzleNo": "1",
              "productNo": "1",
              "tankNo1": "1"
            },
            {
              "nozzleNo": "2",
              "productNo": "2",
              "tankNo1": "2"
            },
            {
              "nozzleNo": "3",
              "productNo": "3",
              "tankNo1": "3"
            },
            {
              "nozzleNo": "4",
              "productNo": "4",
              "tankNo1": "4"
            }
          ]
        }
      ]
    },
    {
      "dispenserID": "2",
      "fuelingPoints": [
        {
          "fuelingPointID": "2",
          "nozzles": [
            {
              "nozzleNo": "1",
              "productNo": "1",
              "tankNo1": "1"
            },
            {
              "nozzleNo": "2",
              "productNo": "2",
              "tankNo1": "2"
            },
            {
              "nozzleNo": "3",
              "productNo": "3",
              "tankNo1": "3"
            },
            {
              "nozzleNo": "4",
              "productNo": "4",
              "tankNo1": "4"
            }
          ]
        }
      ]
    }
  ]
}
```

<b>Response Body:</b>
You get an HTTP Response 200 or 400 depending on whether the request could be processed correctly. 
The Body of the response is the following, where the error property must contain the value “ERRCD_OK” which indicates that the configuration was successful.

```json
{
    "timestamp": "2025-03-21T21:33:23.4991943+00:00",
    "result": "success",
    "error": "ERRCD_OK",
    "message": "Operation completed successfully"
}
```



### Modes

<b>Relative URL:</b> URLAmbiente/v{{Version}}/SiteSystem/SSE/{{SiteSessionId}}/modes </br>
<b>Method:</b> POST </br>
<b>Configuration Type:</b> Optional </br>
<b>Input:</b> application/json </br>
<b>Output:</b> application/json </br>
<b>Use:</b> Allows configuring the operating modes in the session created in the MobileEvents. This is done with the fuelModes object.

<b>Request Body:</b>
```json
{
  "fuelModes":
      [
        {
          "modeNo": "1",
          "modeName": "FullServ"
        },
        {
          "modeNo": "2",
          "modeName": "SelfServ"
        }
      ]
}
```

<b>Response Body:</b>
You get an HTTP Response 200 or 400 depending on whether the request could be processed correctly. 
The Body of the response is the following, where the error property must contain the value “ERRCD_OK” which indicates that the configuration was successful.

```json
{
    "timestamp": "2025-03-21T21:33:23.4991943+00:00",
    "result": "success",
    "error": "ERRCD_OK",
    "message": "Operation completed successfully"
}
```



### Country Settings

<b>Relative URL:</b> URLAmbiente/v{{Version}}/SiteSystem/SSE/{{SiteSessionId}}/CountrySettings </br>
<b>Method:</b> POST </br>
<b>Configuration Type:</b> Opcional </br>
<b>Input:</b> application/json </br>
<b>Output:</b> application/json </br>
<b>Use:</b> Allows to configure in the session created in the MobileEvents, the values of units / language / country code / currency. This is done with the countrySettings object.

<b>Request Body:</b>
```json
{
  "countrySettings":
      {
        "volumeUnit": "GLL",
        "countryCode": "US",
        "language": "eng",
        "localCurrencies":
          [
            "USD"
          ]
      } 
}
```

<b>Response Body:</b>
You get an HTTP Response 200 or 400 depending on whether the request could be processed correctly. 
The Body of the response is the following, where the error property must contain the value “ERRCD_OK” which indicates that the configuration was successful.

```json
{
    "timestamp": "2025-03-21T21:33:23.4991943+00:00",
    "result": "success",
    "error": "ERRCD_OK",
    "message": "Operation completed successfully"
}
```
