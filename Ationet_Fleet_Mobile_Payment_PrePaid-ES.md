![ationetlogo](Content/Images/ATIOnetLogo_250x70.png)
# Ationet Fleet Mobile Payment - PrePaid #

|Información de Documento||
|--- |--- |
|Archivo:|ATIONet - Fleet Mobile Payment PrePaid|
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
- [Mensajería Transaccional](#mensajería-transaccional)
  - [Flujo Transaccional](#flujo-transaccional)
  - [Mensajería del Host al SiteSystem](#mensajería-del-host-al-siteSystem)
    - [Descripción General SSE](#descripción-general-sse)   
    - [FPReserveRequest](#fpreserveRequest)   
    - [AuthorizeRequest](#authorizeRequest)    
    - [FPReserveCancelRequest](#fpreserveCancelRequest)
  - [Mensajería del SiteSystem al Host](#mensajería-del-SiteSystem-al-Host)
     - [Descripción General](#descripción-general)
     - [Reserve Notification POST](#reserve-notification-post)
     - [Reserve Notification DELETE](#reserve-notification-delete)
     - [Transaction Info](#transaction-info)
     - [Authorization Notification](#authorization-notification)
     - [Begin Fueling Notification](#begin-fueling-notification)
     - [Finalize Transaction Notification](#finalize-transaction-notification)
- [Colección Postman](#colección-postman)
- [Ejemplo en C#](#ejemplo-en-c)



## Introducción
Este documento describe la operación para efectuar una transacción con una autorización por medio del autorizador de flota de Ationet a travez de nuestra aplicación móvil.

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


## Mensajería Transaccional


### Flujo Transaccional

<div align="center">
  <img src="https://github.com/user-attachments/assets/4e47c24e-3d12-4cfa-9e7d-b6702c0ab355" alt="Diagrama de Secuencia de Operación">
</div>


### Mensajería del Host al SiteSystem

#### Descripción General SSE
A continuación se detallan todos los mensajes que el SiteSystem podrá recibir mediante el canal SSE (Server-Sent Events), mediante long-lived HTTP connection

#### FPReserveRequest

<b>Output:</b> application/json </br>
<b>Uso:</b> Mediante este evento el Host le notificará al SiteSystem que debe reservar una posición de carga. 
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

<b>Definición de Campos:</b> </br>
<b>id:</b> Corresponde al tipo de mensaje que el servidor le esta notificando al SiteSystem</br>
<b>eventMessage:</b>  Descripción del tipo de mensaje.</br>
<b>timestamp:</b> fecha y hora de la solicitud</br>
<b>UMTI:</b> Id de la transacción en curso</br>
<b>fuelingPointID:</b> Posición que se desea reservar</br>


#### AuthorizeRequest

<b>Output:</b> application/json </br>
<b>Uso:</b> Mediante este evento el Host le notificará al SiteSystem que debe authorizar una posición de carga. Para esto el SiteSystem deberá obtener el detalle de la transacción notificada (UMTI) mediante una solicitud GET (Método trxs/{TransactionId}), a fin de obtener el detalle de la transacción y poder determinar el valor de la operación que se debe autorizar. 
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

<b>Definición de Campos:</b> </br>
<b>id:</b> Corresponde al tipo de mensaje que el servidor le esta notificando al SiteSystem</br>
<b>eventMessage:</b>  Descripción del tipo de mensaje.</br>
<b>timestamp:</b> fecha y hora de la solicitud</br>
<b>UMTI:</b> Id de la transacción en curso</br>


#### FPReserveCancelRequest

<b>Output:</b> application/json </br>
<b>Uso:</b> Mediante este evento el Host le notificará al SiteSystem que posterior a la reserva de la posición de carga ocurrió un error y el flujo no puede avanzar. Por tal motivo el SiteSystem deberá liberar la posición de carga que se encontraba reservada.
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

<b>Definición de Campos:</b> </br>
<b>id:</b> Corresponde al tipo de mensaje que el servidor le esta notificando al SiteSystem</br>
<b>eventMessage:</b>  Descripción del tipo de mensaje.</br>
<b>timestamp:</b> fecha y hora de la solicitud</br>
<b>UMTI:</b> Id de la transacción en curso</br>
<b>fuelingPointID:</b> Posición que se desea liberar</br>



### Mensajería del SiteSystem al Host

#### Descripción General
Esta mensajería es la que deberá implementar el SiteSystem para actuar en consecuencia a las operaciones que se llevan a cabo en base a los mensajes recibidos en el canal SSE. 

#### Reserve Notification (POST)

<b>Relative URL:</b> URLAmbiente/v{{Version}}/SiteSystem/trxs/{{TransactionId}}/FPs/{{FuelPointId}}/reserveNotification</br>
<b>Method:</b> POST </br>
<b>Input:</b> application/json </br>
<b>Output:</b> application/json </br>
<b>Uso:</b> Permite al SiteSystem notificar al Host que la reserva del FuelPoint fue correcta 

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


#### Reserve Notification (DELETE)

<b>Relative URL:</b> URLAmbiente/v{{Version}}/SiteSystem/trxs/{{TransactionId}}/FPs/{{FuelPointId}}/reserveNotification</br>
<b>Method:</b> DELETE </br>
<b>Input:</b> application/json </br>
<b>Output:</b> application/json </br>
<b>Uso:</b> Permite al SiteSystem notificar al Host que no pudo procesar la reserva del FuelPoint 

<b>Request Body:</b>
Vacío

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


#### Transaction Info

<b>Relative URL:</b> URLAmbiente/v{{Version}}/SiteSystem/trxs/{{TransactionId}}</br>
<b>Method:</b> GET </br>
<b>Input:</b> application/json </br>
<b>Output:</b> application/json </br>
<b>Uso:</b> Permite al SiteSystem obtener el detalle de una transacción

<b>Request Body:</b>
Vacío


<b>Response Body:</b>
Se obtiene un HTTP Response 200 o 400 dependiendo si se pudo procesar correctamente el request. 
El SiteSystem podrá obtener el monto autorizado de la propiedad "value" que se encuentra dentro del objeto "finalAmount". "finalAmount" a su vez se encuentra dentro del objeto "paymentInfo".
El Body del response en el caso de obtener un 200, es el siguiente:

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

<b>Relative URL:</b> URLAmbiente/v{{Version}}/SiteSystem/trxs/{{TransactionId}}/authorizationNotification</br>
<b>Method:</b> POST </br>
<b>Input:</b> application/json </br>
<b>Output:</b> application/json </br>
<b>Uso:</b> Permite al SiteSystem notificar al Host que la autorización del FuelPoint fue correcta o incorrecta. 
Para esto deberá modificar el campo "result". 
1. En el caso de que el valor notificado sea "success", el host interpretará que la autorización fue correcta.
2. En el caso de que el valor notificado sea "fail", el host interpretará que la autorización no pudo ser efectuada correctamente por el SiteSystem.


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

<b>Definición de Campos:</b> </br>
<b>result:</b> Corresponde al resultado de la operación. Para este caso, puede ser success o fail</br>
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


#### Begin Fueling Notification

<b>Relative URL:</b> URLAmbiente/v{{Version}}/SiteSystem/trxs/{{TransactionId}}/beginFuelingNotification</br>
<b>Method:</b> POST </br>
<b>Input:</b> application/json </br>
<b>Output:</b> application/json </br>
<b>Uso:</b> Permite al SiteSystem notificar al Host que el despacho pudo comenzar o que existió algún inconveniente. 
Para esto deberá modificar el campo "result". 
1. En el caso de que el valor notificado sea "success", el host interpretará que la autorización fue correcta.
2. En el caso de que el valor notificado sea "fail", el host interpretará que la autorización no pudo ser efectuada correctamente por el SiteSystem.


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

<b>Definición de Campos:</b> </br>
<b>result:</b> Corresponde al resultado de la operación. Para este caso, puede ser success o fail</br>
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


#### Finalize Transaction Notification

<b>Relative URL:</b> URLAmbiente/v{{Version}}/SiteSystem/trxs/{{TransactionId}}/finalizeTrxNotification</br>
<b>Method:</b> POST </br>
<b>Input:</b> application/json </br>
<b>Output:</b> application/json </br>
<b>Uso:</b> Permite al SiteSystem notificar al Host que el despacho pudo comenzar o que existió algún inconveniente. 
Para esto deberá modificar el campo "result". 
1. En el caso de que el valor notificado sea "success", el host interpretará que la autorización fue correcta.
2. En el caso de que el valor notificado sea "fail", el host interpretará que la autorización no pudo ser efectuada correctamente por el SiteSystem.


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

<b>Definición de Campos:</b> </br>
<b>result:</b> Corresponde al resultado de la operación. Para este caso, puede ser success o fail</br>
<b>message:</b>  Descripción del tipo de mensaje.</br>
<b>timestamp:</b> fecha y hora de la solicitud</br>
<b>UMTI:</b> Id de la transacción en curso</br>
<b>FuelingPointId:</b> Posición de carga</br>
<b>FuelCode:</b> Corresponde al Código de producto despachado</br>
<b>productName:</b>  Corresponde al nombre de producto despachado</br>
<b>productCode:</b> Corresponde al Código de producto despachado</br>
<b>Amount:</b> Es el monto despachado</br>
<b>Quantity:</b> Es el volumen despachado</br>
<b>fuelPrice:</b> Es el precio del combustible despachado</br>

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



## Colección Postman

[Link de Descarga de Colección](https://github.com/Ationet/ationetdocs/blob/master/Content/Includes/ANFleetMobilePayment/PrePaid%20-%20Example.postman_collection.json)
</br>
[Link de Descarga de Enviroment](https://github.com/Ationet/ationetdocs/blob/master/Content/Includes/ANFleetMobilePayment/Beta%20-%20MPPAExample.postman_environment.json)
</br>

Nota: Para utilizar la colección se debe solicitar a Ationet que envíe y configure: </br>
URL del ambiente </br>
Usuario </br>
Password </br>
Sitio </br>


## Ejemplo en C#

[Link de Descarga](https://github.com/Ationet/ationetdocs/blob/master/Content/Includes/ANFleetMobilePayment/AtionetMPPAExample.zip)
</br>

Nota: Para utilizar esta solución se debe solicitar a Ationet que envíe y configure: </br>
URL del ambiente </br>
Usuario </br>
Password </br>
Sitio </br>
