## Métodos de Interfaz de Web CheckOut

El web service contiene diferentes métodos donde se ejecutan operaciones de tipo solicitud - respuesta, estas requieren para cada una, parámetros de ingreso que son usados por las estructuras de datos con las que se procesan información para posteriormente retornar una respuesta.

Si deseas ejecutar un método rápidamente puedes usar: [https://dnetix.co/p2p/client](https://dnetix.co/p2p/client)

### CreateRequest

>Ejemplo para la petición de un pago:

```shell
POST /api/session

{
  "auth": {
    "login": "usuarioprueba",
    "tranKey": "jsHJzM3+XG754wXh+aBvi70D9/4=",
    "nonce": "TTJSa05UVmtNR000TlRrM1pqQTRNV1EREprWkRVMU9EZz0=",
    "seed": "2019-04-25T18:17:23-04:00"
  },
    "locale": "en_CO",
    "buyer": {
      "name": "Deion",
      "surname": "Ondricka",
      "email": "dnetix@yopmail.com",
      "document": "1040035000",
      "documentType": "CC",
      "mobile": 3006108300
  },
 
    "payment": {
        "reference": "3210",
        "description": "Pago básico de prueba 04032019",
        "amount": {
            "currency": "COP",
            "total": "10000"
        },
      "allowPartial":false
      },

    "expiration": "2019-03-05T00:00:00-05:00",
    "returnUrl": "https://mysite.com/response/3210",
    "ipAddress": "127.0.0.1",
    "userAgent": "PlacetoPay Sandbox"
}
```
 Solicita la creación de la sesión (petición de cobro o suscripción) y retorna el identificador y la URL de procesamiento. <br> <br>
 
PARÁMETROS

| Name                          | Type                                | Description                         |
|-------------------------------|-------------------------------------|-------------------------------------|
| payload <code>Opcional</code> | [RedirectRequest](#redirectrequest) | Información sobre el pago requerido |

**Retorna** <br>
   **[RedirectResponse](#redirectresponse):** Es un objeto con la información de redirección.

### GetRequestInformation

>Ejemplo para obtener información de una petición:

```shell
POST /api/session/REQUEST_ID
Remplace el REQUEST_ID con el valor devuelto en la respuesta de la petición de pago.

{
  "auth": {
    "login": "usuarioprueba",
    "tranKey": "T0O+x3gNlQUf0iBxEuenPvBPlWs=",
    "nonce": "YzkwODVlODJkZWJiODJiMDk1NTU3OTA5OGJlM2Q3Y2E=",
    "seed": "2019-04-25T18:17:23-04:00",
  }
}
```
Obtiene la información de la sesión y transacciones realizadas. <br> <br> 

PARÁMETROS

| Name                             | Type | Description                             |
|----------------------------------|------|-----------------------------------------|
| requestId <code>Requerido</code> | int  | Identificador de la sesión a consultar. |

**Retorna** <br>
  **[RedirectInformation](#redirectinformation):** Información del estado de la transacción

### ReversePayment

>Ejemplo para revertir un pago aprobado:

```shell
POST /api/reverse
{
    "auth": {
    "login": "usuarioprueba",
    "tranKey": "jsHJzM3+XG754wXh+aBvi70D9/4=",
    "nonce": "TTJSa05UVmtNR000TlRrM1pqQTRNV1EREprWkRVMU9EZz0=",
    "seed": "2019-04-25T18:17:23-04:00"
    },
    "internalReference": "1468647381"
}
```
   Permite revertir un pago aprobado con el código de referencia interna. <br> <br>

PARÁMETROS 

| Name                                     | Type | Description                                                                                                                          |
|------------------------------------------|------|--------------------------------------------------------------------------------------------------------------------------------------|
| internalReference <code>Requerido</code> | int  | Referencia interna de la transacción a reversar que se encuentra en el listado de transacciones del getRequestInformation (payment). |


**Retorna** <br>
**[ReverseResponse](#reverseresponse):** Información de estado de la operación.

### Collect
Permite realizar cobros sin la intervención del usuario usando medios de pago previamente suscritos.<br><br>

PARÁMETROS

| Name                           | Type                              | Description                                                                                                          |
|--------------------------------|-----------------------------------|----------------------------------------------------------------------------------------------------------------------|
| payload <code>Requerido</code> | [CollectRequest](#collectrequest) | Datos del pago, pagador y medio de pago a implementar como se describe en la estructura <code>CollectRequest</code>. |
**Retorna** <br>
**[RedirectInformation](#redirectinformation):** Información de estado de la operación.