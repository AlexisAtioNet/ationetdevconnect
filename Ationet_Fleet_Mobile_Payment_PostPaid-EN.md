![ationetlogo](Content/Images/ATIOnetLogo_250x70.png)
# Ationet Fleet Mobile Payment - PostPaid #

|Document Information||
|--- |--- |
|File:|ATIONet - Fleet Mobile Payment PostPaid|
|Doc Version:|1.0|
|Date:|25-03-2025|
|Author:|Alexis E. Yañez|

|Changes Control|||
|--- |--- |--- |
|Ver.|Date|Changes|
|1.0|25-03-2025|Initial version.|

## Contenido ##
- [Introduction](#Introduction)
- [Authentication](#authentication)
- [State Diagram](#state-diagram)
- [Transactional Messaging](#transactional-messaging)
  - [Host to SiteSystem Messaging](#host-to-siteSystem-messaging)
    - [SSE Overview](#sse-overview)
    - [TransactionDataRequest](#transactionDataRequest)   
    - [CloseTrxRequest](#closeTrxRequest)    
  - [SiteSystem to Host Messaging](#siteSystem-to-Host-messaging)
    - [Overview](#overview)
    - [TransactionData](#transactiondata)
    - [CloseTransactionNotification](#closetransactionnotification)


## Introduction
This document describes the operation for carrying out a transaction using the PostPaid methodology.

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
Authorization: Basic <encoded_credentials>
Example:

Authorization: Basic bm9tYnJlX2RlX3VzdWFyaW86Y29udHJhc2XDsWE=
Make an HTTP Request:

Send the HTTP request with the Authorization header included.
The server will decode the credentials and verify them.
Example HTTP Request

GET /recurso HTTP/1.1
Host: api.ejemplo.com
Authorization: Basic bm9tYnJlX2RlX3VzdWFyaW86Y29udHJhc2XDsWE=


## State Diagram

![Diagrama de Estados - MPPA-PostPago drawio](https://github.com/user-attachments/assets/0715d299-18a9-4cda-ac1b-68f204711bcb)


## Transactional Messaging

### Host to SiteSystem Messaging

#### SSE Overview
Below are all the messages that the SiteSystem can receive through the SSE (Server-Sent Events) channel, using a long-lived HTTP connection.

#### TransactionDataRequest

<b>Output:</b> application/json </br>
<b>Use:</b> Through this event the Host will notify the SiteSystem that it must upload the sale.
<b>Body:</b>
```json
{
  "id": "transactionDataRequest",
  "retry": 1,
  "event": "transactionDataRequest",
  "data": {
        "eventMessage": "fueling point transaction data request event",
        "timestamp": "2009-11-20T17:30:50",
        "UMTI": "1af2f8ee-a656-45e6-8e9c-f2659f94be2f",
        "fuelingPointID": "1"
        }
}
```

<b>Field Definition:</b> </br>
<b>id:</b> Corresponds to the type of message that the server is notifying the SiteSystem about.</br>
<b>eventMessage:</b> Description of the message type.</br>
<b>timestamp:</b> Date and time of the request.</br>
<b>UMTI:</b> ID of the current transaction.</br>
<b>fuelingPointID:</b> Position to be reserved.</br>


#### CloseTrxRequest

<b>Output:</b> application/json </br>
<b>Use:</b> Through this event the Host will notify the SiteSystem that the flow must end.
<b>Body:</b>
```json
{
  "id": "closeTrxRequest",
  "retry": 1,
  "event": "closeTrxRequest",
  "fuelingPointID" = "1",
  "umti" = "1af2f8ee-a656-45e6-8e9c-f2659f94be2f",
  "data": {
        "eventMessage": paymentObject,
        "timestamp": "2009-11-20T17:30:50",
        "UMTI": "1af2f8ee-a656-45e6-8e9c-f2659f94be2f",
        "fuelingPointID" = "1"
        }
}
```

<b>Field Definition:</b> </br>
<b>id:</b> Corresponds to the type of message that the server is notifying the SiteSystem</br>
<b>eventMessage:</b> Payment object of the transaction.</br>
<b>timestamp:</b> Date and time of the request</br>
<b>UMTI:</b> ID of the current transaction</br>
<b>fuelingPointID:</b> Position to be reserved</br>


### SiteSystem to Host Messaging

#### Overview
This messaging is what the SiteSystem must implement to act accordingly to the operations carried out based on the messages received on the SSE channel.

#### TransactionData 

<b>Relative URL:</b> URLEnviroment/v{{Version}}/SiteSystem/trxs/{{TransactionId}}/trxData</br>
<b>Method:</b> POST </br>
<b>Input:</b> application/json </br>
<b>Output:</b> application/json </br>
<b>Use:</b> Allows the SiteSystem to upload the sale that must be paid by the wallet

<b>Request Body:</b>
```json
{
  "umti": "{{TransactionId}}",
  "trxInfo": {
    "timestamp": "2023-07-06T13:47:17.526Z",
    "result": "success",
    "error": "string",
    "message": "string",
    "UMTI": "{{TransactionId}}",
    "transactionStatus": "string",
    "fuelingPointID": "string",
    "merchantID": "string",
    "siteID": {
      "type": "string",
      "id": "string"
    }
  },
  "paymentInfo": {
    "paymentMethod": "string",
    "finalAmount": {
      "value": "string",
      "currency": "string"
    },
    "hostAuthNumber": "string",
    "cardType": "string",
    "cardCircuit": "string"
  },
  "fuelingInfo": {
    "fuelProducts": [
      {
        "productNo": "1",
        "productName": "Combustible 1",
        "productCode": "CODE01",
        "prices": [
          {
            "fuelUnitPrice": {
              "value": "string",
              "currency": "string"
            },
            "priceTier": "string",
            "modeNo": "string"
          }
        ],
        "fuelPrice": {
          "value": "10",
          "currency": "USD"
        },
        "fuelUnitOfMeasurement": "LTR",
        "gradeAllowed": "string",
        "merchandiseCode": "string",
        "taxId": "string"
      }
    ],
    "fuelingPointID": "string",
    "fuelAmount": {
      "value": "1",
      "currency": "USD"
    },
    "quantity": {
      "value": "10",
      "uom": "LTR"
    },
    "serviceLevel": "string",
    "modeNo": "string",
    "priceTier": "string"
  },
  "receiptInfo": [
    "string"
  ]
}
```

<b>Field Definition:</b> </br>
<b>result:</b> Corresponds to the result of the operation. In this case, it can only be success.</br>
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


#### CloseTransactionNotification

<b>Relative URL:</b> URLEnviroment/v{{Version}}/SiteSystem/trxs/{{TransactionId}}/closeTrxNotification</br>
<b>Method:</b> POST </br>
<b>Input:</b> application/json </br>
<b>Output:</b> application/json </br>
<b>Use:</b> Allows the Site System to notify the Host that the sale was completed
<b>Request Body:</b>
Vacío

<b>Response Body:</b>
You will receive an HTTP Response of 200 or 400, depending on whether the request was successfully processed.
The response body is as follows, where the error property must contain the value "ERRCD_OK," indicating that the configuration was successful.

```json
{
  "umti": "{{TransactionId}}",
  "trxInfo": {
    "timestamp": "2023-07-06T14:06:25.148Z",
    "result": "success",
    "error": "string",
    "message": "string",
    "UMTI": "{{TransactionId}}",
    "transactionStatus": "success",
    "fuelingPointID": "string",
    "merchantID": "string",
    "siteID": {
      "type": "string",
      "id": "string"
    }
  },
  "paymentInfo": {
    "paymentMethod": "string",
    "finalAmount": {
      "value": "string",
      "currency": "string"
    },
    "hostAuthNumber": "string",
    "cardType": "string",
    "cardCircuit": "string"
  },
  "fuelingInfo": {
    "fuelProducts": [
      {
        "productNo": "string",
        "productId": {
          "productName": "string",
          "description": "string"
        },
        "productCode": "string",
        "prices": [
          {
            "fuelUnitPrice": {
              "value": "string",
              "currency": "string"
            },
            "priceTier": "string",
            "modeNo": "string"
          }
        ],
        "fuelPrice": {
          "value": "string",
          "currency": "string"
        },
        "fuelUnitOfMeasurement": "string",
        "gradeAllowed": "string",
        "merchandiseCode": "string",
        "taxId": "string"
      }
    ],
    "fuelingPointID": "string",
    "fuelAmount": {
      "value": "string",
      "currency": "string"
    },
    "quantity": {
      "value": "string",
      "uom": "string"
    },
    "serviceLevel": "string",
    "modeNo": "string",
    "priceTier": "string"
  },
  "receiptInfo": [
    "string"
  ]
}
```
