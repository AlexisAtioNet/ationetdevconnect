# ATIONet - Integración de Mercado Pago (MPAv2) #

## Información del documento ##

|Información del documento||
|--- |--- |
|Archivo:| ATIONet - MPAv2 Mercado Pago|
|Version:|1.1|
|Fecha de lanzamiento:|27 de noviembre de 2024|
|Autor:|ATIONet LLC|

|Cambios registrados|||
|--- |--- |--- |
|Ver.|Date|Change Summary|
|1.0|27 de noviembre de 2024|Versión inicial.|
|1.1|27 de noviembre de 2024|Actualización para reflejar el flujo con Mercado Pago.|

## Indice ##

- [Visión General](#visión-general)
	- [Introducción](#introducción)
	- [Entidades Involucradas](#entidades-involucradas)
- [Flujo de Transacciones](#flujo-de-transacciones)
	- [Diagrama de flujo](#diagrama-de-flujo)
	- [Estados de transacción](#estados-de-transacción)
    - [Diagrama de secuencia](#diagrama-de-secuencia)
- [Documentación de la API](#documentación-de-la-api)
	- [Detalles de cada endpoint](#detalles-de-cada-endpoint)
	- [Ejemplos de solicitudes y respuestas](#ejemplos-de-solicitudes-y-respuestas)
- [Manejo de Errores](#manejo-de-errores)
- [MercadoPago API](#mercadopago-api)

</br>

## Visión General

## Introducción

La integración de Mercado Pago para el protocolo MPAv2 permite realizar pagos en estaciones de servicio mediante QR, asegurando que las transacciones sean seguras y cumplan con los estándares 
de autorización exigidos por ATIONet.

## Entidades Involucradas

#### MPA (Mobile Payment Application): Aplicación móvil utilizada por el consumidor para interactuar con el sistema de pagos.
#### MPPA (Mobile Payment Processing Application): Aplicación que gestiona las transacciones y comunica con Mercado Pago.
#### Site System: Sistema local que controla la infraestructura de la estación (ej., dispensadores, terminales).

## Flujo de Transacciones

Diagrama de Flujo
El flujo para post pago con Mercado Pago sigue los pasos ilustrados en el siguiente diagrama:


Estados de Transacción
<table>
    <thead>
        <tr>
            <th>Estado</th>
            <th>Descripción</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>PostPaidCreated</td>
            <td>La transacción se crea tras un PaymentRequest.</td>
        </tr>
        <tr>
            <td>PostPaidLocked</td>
            <td>La bomba queda en estado Busy tras el envío exitoso de TransactionData.</td>
        </tr>
        <tr>
            <td>PostPaidNotFound</td>
            <td>La transacción no fue encontrada tras el envío exitoso de TransactionData.</td>
        </tr>
        <tr>
            <td>PostPaidAccepted</td>
            <td>El pago es aceptado por Mercado Pago.</td>
        </tr>
        <tr>
            <td>PostPaidRejected</td>
            <td>El pago es rechazado por Mercado Pago.</td>
        </tr>
        <tr>
            <td>PostPaidConfirmed</td>
            <td>La transacción es confirmada tras un CloseTrxNotification.</td>
        </tr>
        <tr>
            <td>PostPaidConfirmedFail</td>
            <td>La transacción es confirmada con un fallo tras un CloseTrxNotification.</td>
        </tr>
        <tr>
            <td>CancelledByMPPA</td>
            <td>La transacción es cancelada por timeout.</td>
        </tr>
    </tbody>
</table>


## Diagrama de Secuencia
El flujo típico de una transacción postpago es el siguiente:
1. **Solicitud de pago**: La aplicación móvil (MPA) envía una solicitud de pago al sistema (MPPA). La API recibe los parámetros y genera un identificador único (UMTI). 
La transacción queda en estado PostPaidCreated.
2. **Fusion levanta la venta**: Fusion levanta la venta. Si encuentra la transaction quedara en estado PostPaidLocked y si no la encuentra sera un PostPaidNotFound.
3. **MPPA devuelve el estado del pago**: El MPPA valida el pago, si esta OK la transaction pasa a estado PostPaidAccepted y sino PostPaidRejected.
4. **Fusion nos responde con un CloseNotification**: Una vez aceptado o rechazado el pago, la transaction puede quedar en estado PostPaidConfirmed o PostPaidConfirmedFail.

## Endpoints

### Método PaymentRequest

#### Descripción

El endpoint PaymentRequest inicia una nueva transacción en el flujo post-pago de Mercado Pago. Este método puede ser invocado dos veces en diferentes etapas del flujo:

#### Primera llamada: Si no hay una transacción bloqueada asociada al PosId proporcionado, el sistema creará una nueva transacción y la colocará en el estado PostPaidCreated. 
En esta etapa, la respuesta será un error indicando que la transacción está pendiente de bloqueo.
#### Segunda llamada: Si la transacción ya está en estado PostPaidLocked, el endpoint devolverá la información necesaria para que Mercado Pago complete el proceso de pago.

>Éste método requiere autenticacion a través del encabezado. Deberá ser de tipo basica. ejemplo: `Username y Password`

#### Formato de solicitud

*URL: /v{version}/MPA/mercadopago/{posId}/{walletId}/PaymentRequest* </br>
*Method: HTTPPOST* </br>

##### Encabezados requeridos
<table>
    <thead>
        <tr>
            <th>Nombre</th>
            <th>Descripción</th>
            <th>Ejemplo</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Authorization</td>
            <td>Token de autenticación básica</td>
            <td>Username y Password</td>
        </tr>
        <tr>
            <td>Content-Type</td>
            <td>Tipo de contenido</td>
            <td>application/json</td>
        </tr>
    </tbody>
</table>

##### Descripción de los parámetros

<table>
    <thead>
        <tr>
            <th>Nombre</th>
            <th>Tipo</th>
            <th>Descripción</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>PosId</td>
            <td>String</td>
            <td>Identificador del Punto de venta del FuelPoint. Es requerido</td>
        </tr>
        <tr>
            <td>WalletId</td>
            <td>Guid</td>
            <td>Identificador único de la billetera que inicia la operación. Es requerido</td>
        </tr>
    </tbody>
</table>

#### Formato de respuesta

>Respuesta en la primera llamada (transacción en estado PostPaidCreated)

HTTP Status: 400 Bad Request

```json
{
    "error": {
        "type": "in_process",
        "message": "Transaction pending to lock"
    }
}
```

>Respuesta en la segunda llamada (transacción en estado PostPaidLocked)

HTTP Status: 200 OK

```json
{
    "collector_id": 575883255,
    "sponsor_id": 477574548,
    "items": {
        "title": "MercadoPago",
        "description": "Venta",
        "quantity": 10.00,
        "unitPrice": 10.00,
        "currency": "ARS"
    },
    "external_reference": "ff9b7bc8-3123-4e93-bcc7-00145922ee66",
    "notification_url": "https://localhost:60239/v1/MPA/mercadopago/Notifications/ff9b7bc8-3123-4e93-bcc7-00145922ee66",
    "loyalty": {}
}
```

### Método TransactionData

#### Descripción

Este endpoint permite a Fusion enviar los datos de la transacción a MPPA. Dependiendo del contenido del cuerpo de la solicitud,
el estado de la transacción se actualiza a PostPaidLocked (éxito) o PostPaidNotFound (fallo).

>Éste método requiere autenticacion a través del encabezado. Deberá ser de tipo basica. ejemplo: `Username y Password`

#### Formato de solicitud

*URL: /v{version}/SiteSystem/trxs/{{TransactionId}}/trxData* </br>
*Method: HTTPPOST* </br>

##### Encabezados requeridos
<table>
    <thead>
        <tr>
            <th>Nombre</th>
            <th>Descripción</th>
            <th>Ejemplo</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Authorization</td>
            <td>Token de autenticación básica</td>
            <td>Username y Password</td>
        </tr>
        <tr>
            <td>Content-Type</td>
            <td>Tipo de contenido</td>
            <td>application/json</td>
        </tr>
    </tbody>
</table>

##### Descripción de los parámetros
<table>
    <thead>
        <tr>
            <th>Nombre</th>
            <th>Tipo</th>
            <th>Descripción</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>TransactionId</td>
            <td>Guid</td>
            <td>Identificador único de la transacción. Es requerido</td>
        </tr>
        <tr>
            <td>body</td>
            <td>JSON</td>
            <td>Información de la transacción proporcionada por Fusion. Es requerido</td>
        </tr>
    </tbody>
</table>

#### Body de la solicitud
En este endpoint tenemos la peculiaridad de que es en su body, mas precisamente dentro del "trxInfo" el result es puede variar y contener "success" en el caso de que la transaction
sea procesada correctamente o "failure" para el caso contrario.

```json
{
  "umti": "{{TransactionId}}",
  "trxInfo": {
    "timestamp": "2023-07-06T13:47:17.526Z",
    "result": "success || failure",
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

#### Formato de respuesta

>Respuesta exitosa

HTTP Status: 200 OK

```json
{
    "timestamp": "2024-12-03T13:20:39.5658096-03:00",
    "result": "success",
    "error": "ERRCD_OK",
    "message": "Operation completed successfully"
}
```

>Respuesta con errores comunes

HTTP Status: 400 Bad Request

```json
{
    "timestamp": "2024-12-03T13:52:09.9716595-03:00",
    "result": "success",
    "error": "ERRCD_BADREQUEST",
    "message": "bad request"
}
```

### Método Payment Notification

#### Descripción

Este endpoint se utiliza para notificar a MPPA sobre el resultado de un pago. Dependiendo del status del , 
el estado de la transacción se actualiza como éxito (PostPaidAccepted) o como fallo (PostPaidRejected).

>Éste método requiere autenticacion a través del encabezado. Deberá ser de tipo basica. ejemplo: `Username y Password`

#### Formato de solicitud

*URL: /v{version}/MPA/mercadopago/Notifications/{TransactionId}?topic={topic}&id={NotificationId}* </br>
*Method: HTTPPOST* </br>

##### Encabezados requeridos
<table>
    <thead>
        <tr>
            <th>Nombre</th>
            <th>Descripción</th>
            <th>Ejemplo</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Authorization</td>
            <td>Token de autenticación básica</td>
            <td>Username y Password</td>
        </tr>
        <tr>
            <td>Content-Type</td>
            <td>Tipo de contenido</td>
            <td>application/json</td>
        </tr>
    </tbody>
</table>

##### Descripción de los parámetros
<table>
    <thead>
        <tr>
            <th>Nombre</th>
            <th>Tipo</th>
            <th>Descripción</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>TransactionId</td>
            <td>Guid</td>
            <td>Identificador único de la transacción. Es requerido</td>
        </tr>
        <tr>
            <td>topic</td>
            <td>String</td>
            <td>Es enviado por la billetera. Es requerido</td>
        </tr>
        <tr>
            <td>NotificationId</td>
            <td>String</td>
            <td>Identificador único de la notificación, que vamos a utilizar para ir al host externo para notificarlo. Es requerido</td>
        </tr>
    </tbody>
</table>

#### Formato de respuesta

>Respuesta exitosa

HTTP Status: 200 OK
Para este caso, cuando devolvemos un 200 OK no tenemos una respuesta concreta, pero se envia por el canal SSE una notificacion llamada "closeTrxRequest" que contiene el siguiente mensaje:

```json
{
    "eventMessage": "{\"PaymentType\":\"wallet prueba\",
                      \"PaymentObject\":
                          \"{\\"external_reference\\":null,
                             \\"id\\":\\"91848801230\\",
                             \\"StatementDescriptor\\":
                             \\"CYBERSOURCE * PUMA ENERGY\\",
                             \\"status\\":\\"approved\\",  
                             \\"CurrencyId\\":\\"USD\\",
                             \\"DateCreated\\":\\"2024-05-06T11: 37: 54.3451752Z\\",
                             \\"AuthCode\\":\\"2471778\\",
                             \\"TransactionAmount\\":\\"150\\",
                             \\"CollectorId\\":0,
                             \\"Description\\":null,
                             \\"payment_method_id\\":\\"Otrhers\\",
                             \\"PaymentTypeId\\":\\"ticket\\",
                             \\"Product\\":{\\"Code\\":\\"10087\\",
                             \\"ProductNo\\":\\"2\\"},
                             \\"PuntoVenta\\":null,
                             \\"raw_object\\":null}\"}",
    "UMTI": "102d289f-836e-4679-9ed5-3dacb8f02b99",
    "timestamp": "2024-12-03T15:24:24.8200387-03:00",
    "fuelingPointID": "1"
}
```

>Respuesta con errores comunes

HTTP Status: 400 Bad Request

```json
{
    "timestamp": "2024-12-03T13:52:09.9716595-03:00",
    "result": "success",
    "error": "ERRCD_BADREQUEST",
    "message": "bad request"
}
```

### Método Close Transaction Notification

#### Descripción

Este endpoint permite a Fusion notificar el cierre de una transacción. Dependiendo del valor del campo result en el cuerpo de la solicitud, 
el estado de la transacción se actualiza a PostPaidConfirmed (éxito) o PostPaidConfirmedFail (fallo).

>Éste método requiere autenticacion a través del encabezado. Deberá ser de tipo basica. ejemplo: `Username y Password`

#### Formato de solicitud

*URL: /v{version}/SiteSystem/trxs/{{TransactionId}}/closeTrxNotification* </br>
*Method: HTTPPOST* </br>

##### Encabezados requeridos
<table>
    <thead>
        <tr>
            <th>Nombre</th>
            <th>Descripción</th>
            <th>Ejemplo</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Authorization</td>
            <td>Token de autenticación básica</td>
            <td>Username y Password</td>
        </tr>
        <tr>
            <td>Content-Type</td>
            <td>Tipo de contenido</td>
            <td>application/json</td>
        </tr>
    </tbody>
</table>

##### Descripción de los parámetros
<table>
    <thead>
        <tr>
            <th>Nombre</th>
            <th>Tipo</th>
            <th>Descripción</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>TransactionId</td>
            <td>Guid</td>
            <td>Identificador único de la transacción. Es requerido</td>
        </tr>
        <tr>
            <td>body</td>
            <td>JSON</td>
            <td>Información de la transacción proporcionada por Fusion. Es requerido</td>
        </tr>
    </tbody>
</table>

#### Body de la solicitud
En este endpoint tambien tenemos el detalle de que es en su body, precisamente dentro del "trxInfo" el result es puede variar y contener "success" en el caso de que la transaction
sea procesada correctamente o "fail" para el caso contrario.

```json
{
  "umti": "{{TransactionId}}",
  "trxInfo": {
    "timestamp": "2023-07-06T14:06:25.148Z",
    "result": "fail",
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

#### Formato de respuesta

>Respuesta exitosa

HTTP Status: 200 OK
Y no contiene un body en su respuesta.

>Respuesta con errores comunes

HTTP Status: 400 Bad Request

```json
{
    "timestamp": "2024-12-03T13:52:09.9716595-03:00",
    "result": "success",
    "error": "ERRCD_BADREQUEST",
    "message": "bad request"
}
```

### Manejo de errores

Las salidas exitosas / fallidas en la API de la interfaz se manejarán a través de códigos de estado HTTP.
La solicitud exitosa obtendrá un HTTP 200 y la respuesta resultante.
Si no se procesa la solicitud, se indicará mediante un código de estado de rango HTTP 400. El cuerpo contendrá un solo elemento con formato JSON con los campos 
"ResponseCode", "ResponseMessage" y "ResponseError".


### Ejemplos Comunes de Errores
- **400**: Bad Request.
- **404**: Not found.

## Códigos de Respuesta
### Códigos HTTP
- **200**: Solicitud procesada exitosamente.
- **400**: Error en la solicitud. Revise los datos enviados.
- **500**: Error interno del servidor.

### Ejemplos de Respuesta
#### Respuesta de Error
```json
{
    "timestamp": "2024-12-03T13:52:09.9716595-03:00",
    "result": "success",
    "error": "ERRCD_BADREQUEST",
    "message": "bad request"
}
```

### Mercado Pago API
Mercado Pago API reference: https://www.mercadopago.com.ar/developers/es/guides/instore-payments/qr-payments/qr-gas-station

## Objects references
### Order object
|Field Name|Type|Condition|Descriptions/Field Value(s)|
|---|---|---|---|
|collector_id|Long|Required|Identificador de la cuenta de Mercado Pago a la que se le acreditarán los pagos.|
|sponsor_id|Long|Required|Identificador de una cuenta de Mercado Pago que integra la solución.|
|external_reference|String|Required|Referencia para sincronizar con tu sistema.|
|notification_url|String|Required|URL a la cual se enviarán las notificaciones, definida por el integrador.|
|items|Array|Required|Lista de los productos, donde cada item es un object con los siguientes campos|
|loyalty|Object|Required|Datos necesarios para sumar puntos en un determinado programa de fidelización|

### Items object
|Field Name|Type|Condition|Descriptions/Field Value(s)|
|---|---|---|---|
|title|String|Required|Nombre del producto.|
|quantity|Entero|Required|Cantidad de este producto.|
|unit_price|Decimal|Required|Precio unitario del producto.|
|total_amount|Decimal|Required|Monto total del item.|

### Loyalty object
|Field Name|Type|Condition|Descriptions/Field Value(s)|
|---|---|---|---|
|program|String|Required|Programa de fidelización (serviclub, payback, etc.)|
|transaction_id|String|Required|Número de transacción.|
|invoice_number|String|Required|Número de comprobante.|
|transaction_date|String|Required|Fecha y hora de la transacción (ISO 8601).|
|transaction_amount|Decimal|Required|Importe total de la transacció.|
|store_id|String|Required|Identificador único del negocio (identificador de estación de servicio o APIES).|
|products|Array|Required|Lista de los productos comprados con los siguientes atributos|
|code|String|Required|Código del producto.|
|quantity|Decimal o entero|Required|Por ejemplo 20.50 litros.|
|unit_price|Decimal|Required|Precio unitario del producto.|
|unit|String|Required|Unidad de medida si aplica (litre, etc.)|
|cashier_identification|Object|Required|Datos del empleado|
|type|String|Required|Tipo de documento (DNI, INE, etc.)|
|number|String|Required|Id de documento.|
|period|String|Required|Número del período.|
|shift|String|Required|Número del turno.|
|affinity_plan|String|Required|Plan de afinidad.|


## Conclusión
La implementación de pagos móviles con MercadoPago proporciona una solución robusta para gestionar transacciones en estaciones de servicio. 
Asegúrese de seguir cada paso de configuración y validar las respuestas de la API para garantizar una experiencia fluida para los usuarios.

