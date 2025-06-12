![ationetlogo](Content/Images/ATIOnetLogo_250x70.png)
# Ationet Fleet Mobile Payment - PostPaid #

|Información de Documento||
|--- |--- |
|Archivo:|ATIONet - Fleet Mobile Payment PostPaid|
|Doc Version:|1.0|
|Fecha:|25-03-2025|
|Autor:|Alexis E. Yañez|

|Control de Cambios |||
|--- |--- |--- |
|Ver.|Fecha|Cambios|
|1.0|25-03-2025|Initial version.|

## Contenido ##
- [Introducción](#Introducción)
- [Autenticación](#autenticación)
- [Diagrama de Estados](#diagrama-de-estados)
- [Mensajería Transaccional](#mensajería-transaccional)
  - [Mensajería del Host al SiteSystem](#mensajería-del-host-al-siteSystem)
    - [Descripción General SSE](#descripción-general-sse)   
    - [TransactionDataRequest](#transactionDataRequest)   
    - [CloseTrxRequest](#closeTrxRequest)    
  - [Mensajería del SiteSystem al Host](#mensajería-del-SiteSystem-al-Host)
     - [Descripción General](#descripción-general)
     - [TransactionData](#transactiondata)
     - [CloseTransactionNotification](#closetransactionnotification)


## Introducción
Este documento describe la operación para efectuar una transacción con la metodología PostPaid.

## Autenticación

Pasos para Configurar la Autenticación Básica
Codificación de Credenciales:

Combina el nombre de usuario y la contraseña en un solo string, separados por dos puntos (:).
Codifica este string en Base64.
Ejemplo:

nombre_de_usuario:contraseña

Codificado en Base64:

bm9tYnJlX2RlX3VzdWFyaW86Y29udHJhc2XDsWE=


Agregar el Encabezado de Autorización:

Incluye el string codificado en Base64 en el encabezado de la solicitud HTTP.
El formato del encabezado es:
Authorization: Basic <credenciales_codificadas>
Ejemplo:

Authorization: Basic bm9tYnJlX2RlX3VzdWFyaW86Y29udHJhc2XDsWE=
Realizar la Solicitud HTTP:

Envía la solicitud HTTP con el encabezado de autorización incluido.
El servidor decodificará las credenciales y las verificará.
Ejemplo de Solicitud HTTP

GET /recurso HTTP/1.1
Host: api.ejemplo.com
Authorization: Basic bm9tYnJlX2RlX3VzdWFyaW86Y29udHJhc2XDsWE=


## Diagrama de Estados

![Diagrama de Estados - MPPA-PostPago drawio](https://github.com/user-attachments/assets/0715d299-18a9-4cda-ac1b-68f204711bcb)


## Mensajería Transaccional

### Mensajería del Host al SiteSystem

#### Descripción General SSE
A continuación se detallan todos los mensajes que el SiteSystem podrá recibir mediante el canal SSE (Server-Sent Events), mediante long-lived HTTP connection

#### TransactionDataRequest

<b>Output:</b> application/json </br>
<b>Uso:</b> Mediante este evento el Host le notificará al SiteSystem que debe subir la venta.
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

<b>Definición de Campos:</b> </br>
<b>id:</b> Corresponde al tipo de mensaje que el servidor le esta notificando al SiteSystem</br>
<b>eventMessage:</b>  Descripción del tipo de mensaje.</br>
<b>timestamp:</b> fecha y hora de la solicitud</br>
<b>UMTI:</b> Id de la transacción en curso</br>
<b>fuelingPointID:</b> Posición que se desea reservar</br>


#### CloseTrxRequest

<b>Output:</b> application/json </br>
<b>Uso:</b> Mediante este evento el Host le notificará al SiteSystem que debe finalizar el flujo.
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

<b>Definición de Campos:</b> </br>
<b>id:</b> Corresponde al tipo de mensaje que el servidor le esta notificando al SiteSystem</br>
<b>eventMessage:</b>  Objeto pago de la transacción.</br>
<b>timestamp:</b> fecha y hora de la solicitud</br>
<b>UMTI:</b> Id de la transacción en curso</br>
<b>fuelingPointID:</b> Posición que se desea reservar</br>


### Mensajería del SiteSystem al Host

#### Descripción General
Esta mensajería es la que deberá implementar el SiteSystem para actuar en consecuencia a las operaciones que se llevan a cabo en base a los mensajes recibidos en el canal SSE. 

#### TransactionData 

<b>Relative URL:</b> URLAmbiente/v{{Version}}/SiteSystem/trxs/{{TransactionId}}/trxData</br>
<b>Method:</b> POST </br>
<b>Input:</b> application/json </br>
<b>Output:</b> application/json </br>
<b>Uso:</b> Permite al SiteSystem subir la venta que debe ser abonada por la billetera

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

<b>Definición de Campos:</b> </br>
<b>result:</b> Corresponde al resultado de la operación. Para este caso solo puede ser success</br>
<b>message:</b>  Descripción del tipo de mensaje.</br>
<b>timestamp:</b> fecha y hora de la solicitud</br>
<b>UMTI:</b> Id de la transacción en curso</br>
<b>FuelingPointId:</b> Posición que se desea liberar</br>


<b>Response Body:</b>
Se obtiene un HTTP Response 200 o 400 dependiendo si se pudo procesar correctamente el request. 
El Body del response es el siguiente, donde la propiedad error debe contener el valor "ERRCD_OK" el cual indica que la configuración se pudo realizar de forma correcta.

```json
{
    "timestamp": "2025-03-21T21:33:23.4991943+00:00",
    "result": "success",
    "error": "ERRCD_OK",
    "message": "Operation completed successfully"
}
```


#### CloseTransactionNotification

<b>Relative URL:</b> URLAmbiente/v{{Version}}/SiteSystem/trxs/{{TransactionId}}/closeTrxNotification</br>
<b>Method:</b> POST </br>
<b>Input:</b> application/json </br>
<b>Output:</b> application/json </br>
<b>Uso:</b> Permite al SiteSystem notificar al Host que pudo finalizar la venta
<b>Request Body:</b>
Vacío

<b>Response Body:</b>
Se obtiene un HTTP Response 200 o 400 dependiendo si se pudo procesar correctamente el request. 
El Body del response es el siguiente, donde la propiedad error debe contener el valor "ERRCD_OK" el cual indica que la configuración se pudo realizar de forma correcta.

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
