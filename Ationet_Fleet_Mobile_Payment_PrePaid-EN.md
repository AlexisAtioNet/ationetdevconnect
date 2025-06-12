![ationetlogo](Content/Images/ATIOnetLogo_250x70.png)
# Ationet Fleet Mobile Payment - PrePaid #

|Document Information||
|--- |--- |
|Archivo:|ATIONet - Fleet Mobile Payment PrePaid|
|Doc Version:|1.0|
|Date:|25-03-2025|
|Author:|Alexis E. Yañez|

|Change Control |||
|--- |--- |--- |
|Ver.|Date|Changes|
|1.0|25-03-2025|Initial version.|

## Content ##
- [Introduction](#introduction)
- [Authentication](#authentication)
- [Diagram State](#diagram-state)
- [Transactional Messaging](#transactional-messaging)
  - [Transactional Flow](#transactional-flow)
  - [Host to Site System Messaging](#host-to-site-system-messaging)
    - [SSE General Description](#sse-general-description)   
    - [FPReserveRequest](#fpreserveRequest)   
    - [AuthorizeRequest](#authorizeRequest)    
    - [FPReserveCancelRequest](#fpreserveCancelRequest)
  - [Site System at Host Messaging](#site-system-at-host-messaging)
     - [General Description](#general-description)
     - [Reserve Notification POST](#reserve-notification-post)
     - [Reserve Notification DELETE](#reserve-notification-delete)
     - [Transaction Info](#transaction-info)
     - [Authorization Notification](#authorization-notification)
     - [Begin Fueling Notification](#begin-fueling-notification)
     - [Finalize Transaction Notification](#finalize-transaction-notification)
- [Postman Collection](#postman-collection)
- [Example in C#](#example-in-c)



## Introduction
This document describes how to complete a transaction with authorization through the Ationet Fleet Authorizer via our mobile app.

## Authentication

Steps to Configure Basic Authentication
Credential Encoding:

Combine the username and password into a single string, separated by a colon (:).
Encode this string in Base64.
Example:

username:password

Base64 encoded:

bm9tYnJlX2RlX3VzdWFyaW86Y29udHJhc2XDsWE=


Add the Authorization Header:

Includes the Base64-encoded string in the HTTP request header.
The header format is:
Authorization: Basic <credenciales_codificadas>
Ejemplo:

Authorization: Basic bm9tYnJlX2RlX3VzdWFyaW86Y29udHJhc2XDsWE=
Make an HTTP Request:

Send the HTTP request with the Authorization header included.
The server will decode the credentials and verify them.
Example HTTP Request

GET /recurso HTTP/1.1
Host: api.ejemplo.com
Authorization: Basic bm9tYnJlX2RlX3VzdWFyaW86Y29udHJhc2XDsWE=


## Diagram State

![Diagrama de Estados - MPPA-PrePago drawio](https://github.com/user-attachments/assets/12ba858c-e6fb-4176-bf38-ce5286a76689)


## Transactional Messaging


### Transactional Flow

<div align="center">
  <img src="https://github.com/user-attachments/assets/4e47c24e-3d12-4cfa-9e7d-b6702c0ab355" alt="Diagrama de Secuencia de Operación">
</div>


### Host to Site System Messaging

#### SSE General Description
Below are all the messages that the SiteSystem can receive through the SSE (Server-Sent Events) channel, using a long-lived HTTP connection.

#### FPReserveRequest

<b>Output:</b> application/json </br>
<b>Use:</b> Through this event the Host will notify the SiteSystem that it must reserve a loading position.
<b>Body:</b>
```json
{
  "id": "FPReserveRequest",
  "retry": 1,
  "event": "FPReserveRequest",
  "data": {
        "eventMessage": "fueling point reserve request event",
        "timestamp": "2009-11-20T17:30:50",
        "UMTI": "1af2f8ee-a656-45e6-8e9c-f2659f94be2f",
        "fuelingPointID": "1"
        }
}
```

<b>Definition of Fields:</b> </br>
<b>id:</b> Corresponds to the type of message that the server is notifying the SiteSystem</br>
<b>eventMessage:</b>  Description of the message type.</br>
<b>timestamp:</b> date and time of the request</br>
<b>UMTI:</b> ID of the current transaction</br>
<b>fuelingPointID:</b> Position to be reserved</br>


#### AuthorizeRequest

<b>Output:</b> application/json </br>
<b>Use:</b> Through this event, the Host will notify the SiteSystem that it must authorize a charge position. To do this, the SiteSystem must obtain the details of the notified transaction (UMTI) via a GET request (trxs/{TransactionId} method) in order to obtain the transaction details and determine the value of the transaction to be authorized. 
<b>Body:</b>
```json
{
  "id": "authorizeRequest",
  "retry": 1,
  "event": "authorizeRequest",
  "data": {
        "eventMessage": "authorize request event",
        "timestamp": "2009-11-20T17:30:50",
        "UMTI": "1af2f8ee-a656-45e6-8e9c-f2659f94be2f"
        }
}
```

<b>Definition of Fields:</b> </br>
<b>id:</b> Corresponds to the type of message that the server is notifying the SiteSystem</br>
<b>eventMessage:</b>  Description of the message type.</br>
<b>timestamp:</b> date and time of the request</br>
<b>UMTI:</b> ID of the current transaction</br>


#### FPReserveCancelRequest

<b>Output:</b> application/json </br>
<b>Use:</b> Through this event, the Host will notify the SiteSystem that an error occurred after reserving the loading slot and the flow cannot proceed. Therefore, the SiteSystem must release the reserved loading slot.
<b>Body:</b>
```json
{
  "id": "FPReserveCancelRequest",
  "retry": 1,
  "event": "FPReserveCancelRequest",
  "data": {
        "eventMessage": "authorize request event",
        "timestamp": "2009-11-20T17:30:50",
        "UMTI": "1af2f8ee-a656-45e6-8e9c-f2659f94be2f",
        "fuelingPointID": "1"
        }
}
```

<b>Definition of Fields:</b> </br>
<b>id:</b> Corresponds to the type of message that the server is notifying the SiteSystem</br>
<b>eventMessage:</b>  Description of the message type.</br>
<b>timestamp:</b> date and time of the request</br>
<b>UMTI:</b> ID of the current transaction</br>
<b>fuelingPointID:</b> Position to be reserved</br>



### Site System at Host Messaging

#### General Description
This messaging is what the SiteSystem must implement to act accordingly to the operations carried out based on the messages received on the SSE channel.

#### Reserve Notification (POST)

<b>Relative URL:</b> URLEnviroment/v{{Version}}/SiteSystem/trxs/{{TransactionId}}/FPs/{{FuelPointId}}/reserveNotification</br>
<b>Method:</b> POST </br>
<b>Input:</b> application/json </br>
<b>Output:</b> application/json </br>
<b>Use:</b> Allows the Site System to notify the Host that the FuelPoint reservation was successful

<b>Request Body:</b>
```json
{
    "timestamp": "2023-06-02T12:20:50.311Z",
    "result": "success",
    "error": "00000",
    "message": "THE FUELING POINT WAS RESERVED SUCCESSFULLY",
    "UMTI": "{{TransactionId}}",
    "TransactionStatus": "PUMPRESERVED",
    "FuelingPointId": "{{FuelPointId}}",
    "MerchantId": "0192-7509",
    "SiteId": {
        "Type": "SAP",
        "Id": "{{SiteCode}}"
        }
}
```

<b>Definition of Fields:</b> </br>
<b>result:</b> Corresponds to the result of the operation. In this case, it can only be success.</br>
<b>message:</b>  Description of the message type.</br>
<b>timestamp:</b> date and time of the request</br>
<b>UMTI:</b> ID of the current transaction</br>
<b>FuelingPointId:</b> Position to be released</br>


<b>Response Body:</b>
You will receive an HTTP Response of 200 or 400, depending on whether the request was successfully processed.
The response body is as follows, where the error property must contain the value "ERRCD_OK," indicating that the configuration was successful.

```json
{
    "timestamp": "2025-03-21T21:33:23.4991943+00:00",
    "result": "success",
    "error": "ERRCD_OK",
    "message": "Operation completed successfully"
}
```


#### Reserve Notification (DELETE)

<b>Relative URL:</b> URLEnviroment/v{{Version}}/SiteSystem/trxs/{{TransactionId}}/FPs/{{FuelPointId}}/reserveNotification</br>
<b>Method:</b> DELETE </br>
<b>Input:</b> application/json </br>
<b>Output:</b> application/json </br>
<b>Use:</b> Allows the SiteSystem to notify the Host that it was unable to process the FuelPoint reservation 

<b>Request Body:</b>
Empty

<b>Response Body:</b>
You will receive an HTTP Response of 200 or 400, depending on whether the request was successfully processed.
The response body is as follows, where the error property must contain the value "ERRCD_OK," indicating that the configuration was successful.

```json
{
    "timestamp": "2025-03-21T21:33:23.4991943+00:00",
    "result": "success",
    "error": "ERRCD_OK",
    "message": "Operation completed successfully"
}
```


#### Transaction Info

<b>Relative URL:</b> URLEnviroment/v{{Version}}/SiteSystem/trxs/{{TransactionId}}</br>
<b>Method:</b> GET </br>
<b>Input:</b> application/json </br>
<b>Output:</b> application/json </br>
<b>Use:</b> Allows the SiteSystem to obtain the details of a transaction

<b>Request Body:</b>
Empty


<b>Response Body:</b>
You will receive a 200 or 400 HTTP Response, depending on whether the request was successfully processed.
The SiteSystem can obtain the authorized amount from the "value" property located within the "finalAmount" object. "finalAmount" is in turn located within the "paymentInfo" object.
The response body, if you receive a 200, is as follows:

```json
{
    "trxInfo": {
        "timestamp": "2025-01-30T17:48:53.3941524+00:00",
        "result": "success",
        "error": "ERRCD_OK",
        "message": "Complete",
        "UMTI": "7a74d97d-d7e0-41d7-871a-bfe89a30babb",
        "transactionStatus": "authorized",
        "fuelingPointID": "1",
        "merchantID": "",
        "siteID": {
            "type": "",
            "id": "15241"
        }
    },
    "paymentInfo": {
        "paymentMethod": null,
        "finalAmount": {
            "value": "3.00",
            "currency": null
        },
        "hostAuthNumber": "004745154",
        "cardType": null,
        "cardCircuit": null
    },
    "fuelingInfo": {
        "fuelProducts": [
            {
                "productNo": null,
                "productName": null,
                "productCode": "1",
                "prices": null,
                "fuelPrice": {
                    "value": "3.00",
                    "currency": "USD"
                },
                "fuelUnitOfMeasurement": null,
                "gradeAllowed": "yes",
                "merchandiseCode": null,
                "taxId": null
            }
        ],
        "fuelingPointID": "1",
        "fuelAmount": {
            "value": "3.00",
            "currency": "USD"
        },
        "quantity": {
            "value": "1",
            "uom": null
        },
        "serviceLevel": "full",
        "modeNo": "1",
        "priceTier": "credit"
    },
    "customerPreferences": {
        "receipt": null
    }
}
```



#### Authorization Notification

<b>Relative URL:</b> URLEnviroment/v{{Version}}/SiteSystem/trxs/{{TransactionId}}/authorizationNotification</br>
<b>Method:</b> POST </br>
<b>Input:</b> application/json </br>
<b>Output:</b> application/json </br>
<b>Use:</b> Allows the SiteSystem to notify the Host whether the FuelPoint authorization was successful or unsuccessful.
To do this, you must modify the "result" field.
1. If the reported value is "success," the Host will interpret the authorization as successful.
2. If the reported value is "fail," the Host will interpret the authorization as unsuccessful.


<b>Request Body:</b>
```json
{ 
    "timestamp": "2023-06-02T15:36:50.311Z", 
    "result": "success", 
    "error": "00000", 
    "message": "the fueling point was approved successfully", 
    "UMTI": "{{TransactionId}}", 
    "transactionStatus": "authorized", 
    "fuelingPointID": "{{FuelPointId}}", 
    "merchantID": "0192-7509", 
    "siteID": 
        { 
            "type": "SAP", 
            "id": "{{SiteCode}}" 
        } 
}
```

<b>Field Definition:</b> </br>
<b>result:</b> Corresponds to the result of the operation. In this case, it can be success or fail.</br>
<b>message:</b> Description of the message type.</br>
<b>timestamp:</b> Date and time of the request.</br>
<b>UMTI:</b> ID of the current transaction.</br>
<b>FuelingPointId:</b> Position to be released.</br>


<b>Response Body:</b>
You will receive an HTTP Response of 200 or 400, depending on whether the request was successfully processed.
The response body is as follows, where the error property must contain the value "ERRCD_OK," indicating that the configuration was successful.

```json
{
    "timestamp": "2025-03-21T21:33:23.4991943+00:00",
    "result": "success",
    "error": "ERRCD_OK",
    "message": "Operation completed successfully"
}
```


#### Begin Fueling Notification

<b>Relative URL:</b> URLEnviroment/v{{Version}}/SiteSystem/trxs/{{TransactionId}}/beginFuelingNotification</br>
<b>Method:</b> POST </br>
<b>Input:</b> application/json </br>
<b>Output:</b> application/json </br>
<b>Use:</b> Allows the SiteSystem to notify the Host that dispatch was able to begin or that there was a problem.
To do this, you must modify the "result" field.
1. If the reported value is "success," the Host will interpret the authorization as successful.
2. If the reported value is "fail," the Host will interpret the authorization as unsuccessful.

<b>Request Body:</b>
```json
{
    "timestamp": "2023-06-02T13:36:50.311Z",
    "result": "success",
    "error": "00000",
    "message": "the fueling point is fueling",
    "UMTI": "{{TransactionId}}",
    "transactionStatus": "beginFueling",
    "fuelingPointID": "{{FuelPointId}}",
    "fuelingPointState": "FUELING",
    "merchantID": "0192-7509",
    "siteID": {
        "type": "SAP",
        "id": "{{SiteCode}}"
        }
}
```

<b>Field Definition:</b> </br>
<b>result:</b> Corresponds to the result of the operation. In this case, it can be success or fail.</br>
<b>message:</b> Description of the message type.</br>
<b>timestamp:</b> Date and time of the request.</br>
<b>UMTI:</b> ID of the current transaction.</br>
<b>FuelingPointId:</b> Position to be released.</br>


<b>Response Body:</b>
You will receive an HTTP Response of 200 or 400, depending on whether the request was successfully processed.
The response body is as follows, where the error property must contain the value "ERRCD_OK," indicating that the configuration was successful.

```json
{
    "timestamp": "2025-03-21T21:33:23.4991943+00:00",
    "result": "success",
    "error": "ERRCD_OK",
    "message": "Operation completed successfully"
}
```


#### Finalize Transaction Notification

<b>Relative URL:</b> URLEnviroment/v{{Version}}/SiteSystem/trxs/{{TransactionId}}/finalizeTrxNotification</br>
<b>Method:</b> POST </br>
<b>Input:</b> application/json </br>
<b>Output:</b> application/json </br>
<b>Use:</b> Allows the SiteSystem to notify the Host that dispatch was able to begin or that there was a problem.
To do this, you must modify the "result" field.
1. If the reported value is "success," the Host will interpret the authorization as successful.
2. If the reported value is "fail," the Host will interpret the authorization as unsuccessful.


<b>Request Body:</b>
```json
{
    "TrxInfo": { 
        "timestamp": "2023-06-02T15:36:50.311Z", 
        "result": "success", 
        "error": "00000", 
        "message": "the fueling point was approved successfully", 
        "UMTI": "{{TransactionId}}", 
        "transactionStatus": "finalized", 
        "fuelingPointID": "{{FuelPointId}}", 
        "merchantID": "0192-7509", 
        "siteID": 
            { 
                "type": "SAP", 
                "id": "1234" 
            } 
    },
    "paymentInfo": {
        "cardCircuit": "MCB",
        "paymentMethod": "credit",
        "finalAmount": {
            "value": "1",
            "currency": null
        },
        "hostAuthNumber": "312350",
        "cardType": "MASTERCARD"
        },
    "fuelingInfo": {
        "fuelProducts": [{
        "productNo": "{{FuelCode}}",
        "productName": "{{FuelCode}}",
        "productCode": "{{FuelCode}}",
        "fuelPrice": {"value":"{{fuelPrice}}"},
        "fuelUnitOfMeasurement": "GLL",
        "gradeAllowed": "true"
      }],
        "fuelingPointID": "{{fuelingPointId}}",
        "fuelAmount": {
            "value": "{{Amount}}",
            "currency": null
        },
        "quantity": {
            "value": "{{Quantity}}",
            "uom": "l"
        },
        "serviceLevel": "full",
        "modeNo": "1"
        },
    "receiptInfo": [
    "WELCOME",
    "IBERA 2141 CABA",
    "ORIONTECH S.A.",
    "CUIT 30-12331129-2",
    "DATE 09/07/16  12:29",
    "TRAN# 9030038",
    "FP# 02",
    "SERVICE LEVEL: FullServ",
    "PRODUCT: PLUS",
    "GALLONS: 0.926",
    "PRICE/G: $ 2.159",
    "FUEL SALE  $ 2.00",
    "THANK YOU",
    "HAVE A NICE DAY"
    ]
}
```

<b>Field Definition:</b> </br>
<b>result:</b> Corresponds to the result of the operation. In this case, it can be success or fail.
<b>message:</b> Description of the message type.</br>
<b>timestamp:</b> Date and time of the request.</br>
<b>UMTI:</b> ID of the current transaction.</br>
<b>FuelingPointId:</b> Loading position.</br>
<b>FuelCode:</b> Corresponds to the product code dispatched.</br>
<b>productName:</b> Corresponds to the name of the product dispatched.</br>
<b>productCode:</b> Corresponds to the product code dispatched.</br>
<b>Amount:</b> The amount dispatched.</br>
<b>Quantity:</b> The volume dispatched.</br>
<b>fuelPrice:</b> The price of the fuel dispatched.

<b>Response Body:</b>
You will receive an HTTP Response of 200 or 400, depending on whether the request was successfully processed.
The response body is as follows, where the error property must contain the value "ERRCD_OK," indicating that the configuration was successful.

```json
{
    "timestamp": "2025-03-21T21:33:23.4991943+00:00",
    "result": "success",
    "error": "ERRCD_OK",
    "message": "Operation completed successfully"
}
```



## Postman Collection

[Collection Download Link](https://github.com/Ationet/ationetdocs/blob/master/Content/Includes/ANFleetMobilePayment/PrePaid%20-%20Example.postman_collection.json)
</br>
[Environment Download Link](https://github.com/Ationet/ationetdocs/blob/master/Content/Includes/ANFleetMobilePayment/Beta%20-%20MPPAExample.postman_environment.json)
</br>

Note: To use the collection, you must request Ationet to send and configure the following: </br>
Environment URL </br>
User </br>
Password </br>
Site </br>


## Example in C#

[Download Link](https://github.com/Ationet/ationetdocs/blob/master/Content/Includes/ANFleetMobilePayment/AtionetMPPAExample.zip)
</br>

Note: To use this solution, you must request Ationet to send and configure the following: </br>
Environment URL </br>
User </br>
Password </br>
Site </br>
