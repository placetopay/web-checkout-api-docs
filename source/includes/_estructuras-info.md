# Estructuras de información
Esta sección describe cada una de las estructuras de datos usada por los métodos de web service.

## RedirectRequest

> Ejmeplo de una estructura RedirectRequest para la solicitud de un pago:

```shell
    ...
       {
        "locale": "es_CO",
        "buyer": {
            "name": "Prof. General Hoppe",
            "surname": "Kuhic",
            "email": "ejohns@yahoo.com",
            "documentType": "CC",
            "document": "4216523744",
            "mobile": "3006108300",
            "address": {
                "street": "27791 Kyle Vista",
                "city": "Metzton",
                "state": "Antioquia",
                "postalCode": "46366-1132",
                "country": "US",
                "phone": "438-475-0498 x513"
            }
        },
        "payment": {
            "reference": "TEST_20190517_161956",
            "description": "Et alias quia nisi sequi.",
            "amount": {
            "taxes": [
                {
                "kind": "ice",
                "amount": 2.76,
                "base": 23
                },
                {
                "kind": "valueAddedTax",
                "amount": 4.37,
                "base": 23
                }
            ],
            "details": [
                {
                "kind": "shipping",
                "amount": 1.15
                },
                {
                "kind": "tip",
                "amount": 1.15
                },
                {
                "kind": "subtotal",
                "amount": 23
                }
            ],
            "currency": "USD",
            "total": 32.43
            },
            "items": [
            {
                "sku": 62762,
                "name": "Placeat consequuntur.",
                "category": "physical",
                "qty": 1,
                "price": 23,
                "tax": 4.37
            }
            ],
            "recurring_remove": {
            "periodicity": "D",
            "interval": 7,
            "nextPayment": "2019-05-19",
            "maxPeriods": 4,
            "notificationUrl": "https://dnetix.co/ping/recurring_notification"
            },
            "shipping": {
            "name": "Prof. General Hoppe",
            "surname": "Kuhic",
            "email": "ejohns@yahoo.com",
            "documentType": "CC",
            "document": "4216523744",
            "mobile": "3006108300",
            "address": {
                "street": "27791 Kyle Vista",
                "city": "Metzton",
                "state": "Antioquia",
                "postalCode": "46366-1132",
                "country": "US",
                "phone": "438-475-0498 x513"
            }
            },
            "allowPartial": false
        },
        "expiration": "2019-05-18T16:19:56-05:00",
        "ipAddress": "190.85.82.130",
        "userAgent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/74.0.3729.131 Safari/537.36",
        "returnUrl": "https://dnetix.co/p2p/client",
        "cancelUrl": "https://dnetix.co",
        "skipResult": false,
        "noBuyerFill": false,
        "captureAddress": false,
        "paymentMethod": null,
        "fields": [
            {
            "keyword": "Redeem Code",
            "value": 226931,
            "displayOn": "payment"
            }
        ],
    ...    
```
   Estructura que contiene toda la información acerca de la transacción para ser procesada. <br><br>

ATRIBUTOS

Nombre | Tipo | Descripción
-------|------|------------
locale <code>Opcional</code> | string | Definido con los códigos ISO 639 (language) y ISO 3166-1 alpha-2 (2-letras del país). <code>ej. en_US, es_CO</code>
payer <code>Opcional</code> | [Person](#person) | Información del ordenante, si establece este objeto, los datos del pagador utilizarán esta información.
buyer <code>Opcional</code> | [Person](#person) | Información del comprador en la transacción.
payment <code>Opcional</code> | [PaymentRequest](#paymentrequest) | Objeto de pago cuando necesite solicitar un cobro.
subscription <code>Opcional</code> | [SubscriptionRequest](#subscriptionrequest) | Objeto de suscripción utilizado cuando se necesita un token.
fields <code>Opcional</code> | [NameValuePair[]](#namevaluepair) | Información adicional relacionada con la solicitud que el comercio requiere para guardar con la transacción.
paymentMethod <code>Opcional</code> | string | Forzar el medio de pago en la interfaz de redirección, los códigos aceptados son los de la lista. Si necesita más de uno separarlos con coma. <code>ej. _ATH_,_PSE_,CR_VS</code>
expiration <code>Requerido</code> | dateTime | Fecha de expiración de una solicitud, el usuario debe terminar el proceso antes de esta fecha. i.e. 2019-03-05T00:00:00-05:00.
returnUrl <code>Requerido</code> | string | URL de retorno es utilizada para redirigir al usuario una vez termina la operación de pago.
cancelUrl | string | URL para retornar cuando el cliente aborte la operación.
ipAddress <code>Requerido</code> | string | Dirección IP del usuario.
userAgent <code>Requerido</code> | string | Agente de usuario informado por el usuario.
skipResult <code>Opcional</code> | bool | Si se envía el parámetro como true se omite la pantalla del resultado en redirección y una vez aprobado el pago regresa al comercio automaticamente.
noBuyerFill <code>Opcional</code> | bool | Por defecto se llenan los datos que se piden en redirección con los del comprador (buyer) enviados, si se envía como true se omite este autocompletado automático.

**Retorna:** [RedirectResponse](#redirectresponse)

## RedirectResponse
     
   > Ejemplo de una estructura RedirectResponse con respuesta aprobada en una solicitud de pago con autenticación:

```shell
{
    "status": {
        "status": "OK",
        "reason": "PC",
        "message": "La petición se ha procesado correctamente",
        "date": "2019-03-04T16:50:02-05:00"
    },
    "requestId": 181348,
    "processUrl": "https://test.placetopay.com/redirection/session/181348/43d83d36aa46de5f993aafb9b3e0be48"
}
```
 Estructura que contiene la respuesta inicial desde el método [createRequest](#createrequest). <br><br>

ATRIBUTOS

Parametro | Tipo | Descripción
--------- | ---- | -----------
status <code>Requerido</code> | <a href="#status">Status</a> | Estructura que contiene el estado de la solicitud de pago. 
requestId <code>Opcional</code> | Int | Referencia única de la sesión de pago.
processUrl <code>Opcional</code> | String | URL donde se redirecciona al usuario para completar el proceso de pago.

## RedirectInformation
   Estructura de respuesta a una solicitud para una información de transacción. <br><br>

ATRIBUTOS

Nombre | Tipo | Descripción
-------|------|------------
status <code>Requerido</code> | <a href="#status">Status</a> | Estado de esta solicitud, debe observar el estado interno de cada objeto.
request <code>Opcional</code> | [RedirectRequest](#redirectrequest) | Información con la solicitud original.
payment <code>Opcional</code> | [Transaction[]](#transaction) | Información relacionada con el pago si este fue solicitado.
subscription <code>Opcional</code> | [SubscriptionResponse](#subscriptionresponse) | Información relacionado con la suscripción si esta fue solicitada.

## ReverseResponse
   Estructura de respuesta a una solicitud de pago reversado. <br><br>

ATRIBUTOS

Nombre | Tipo | Descripción
-------|------|------------
status <code>Requerido</code> | <a href="#status">Status</a> | Estado de la solicitud será APROBADO si se ha realizado el reverso de lo contrario puede ser RECHAZADA.
payment <code>Opcional</code> | [Transaction](#transaction) | Si el reverso fue exitoso, se almacena como una nueva transacción.

## Person
   Estructura que refleja la información de una persona involucrada en una transacción. <br><br>

ATRIBUTOS

Parametro | Tipo | Descripción
--------- | ---- | -----------
documentType <code>Opcional</code> | <a href="#tipos-de-documento">DocumentType</a> | Tipo de identificación del usuario pagador.
document | String | Identificación.
name <code>Requerido</code> | String | Nombres de la persona.
surname <code>Opcional</code> | String | Apellidos de la persona.
company <code>Opcional</code> | String | Nombre de la empresa en la que trabaja o representa.
email <code>Requerido</code> | String | Correo electrónico de la persona.
address <code>Opcional</code> | <a href="#address">Address</a> | Información completa de la dirección.
mobile <code>Opcional</code> | [PhoneNumberType](#phonenumbertype) | Número celular, el <code>PhoneNumberType</code> restringe la longitud del número de teléfono de la cadena a 30 caracteres.

## PaymentRequest
  
> Ejemplo de una estructura payment para la solicitud de un pago:

```shell
   ...
    "payment": {
            "reference": "TEST_20190517_135449",
            "description": "Et est suscipit fugit possimus.",
            "amount": {
            "taxes": [
                {
                "kind": "ice",
                "amount": 1.56,
                "base": 13
                },
                {
                "kind": "valueAddedTax",
                "amount": 2.47,
                "base": 13
                }
            ],
            "details": [
                {
                "kind": "shipping",
                "amount": 0.65
                },
                {
                "kind": "tip",
                "amount": 0.65
                },
                {
                "kind": "subtotal",
                "amount": 13
                }
            ],
            "currency": "USD",
            "total": 18.33
            },
            "items": [
            {
                "sku": 73131,
                "name": "Veniam quia.",
                "category": "physical",
                "qty": 1,
                "price": 13,
                "tax": 2.47
            }
            ],
            "recurring_remove": {
            "periodicity": "D",
            "interval": 7,
            "nextPayment": "2019-05-19",
            "maxPeriods": 4,
            "notificationUrl": "https://dnetix.co/ping/recurring_notification"
            },
            "shipping": {
            "name": "Miss Fanny Jacobson",
            "surname": "Turcotte",
            "email": "lisandro69@yahoo.com",
            "documentType": "CC",
            "document": "8792527177",
            "mobile": "3006108300",
            "address": {
                "street": "44669 Lowe Lakes Apt. 468",
                "city": "Port Emileview",
                "state": "Antioquia",
                "postalCode": "00555",
                "country": "US",
                "phone": "1-868-319-9048"
            }
            },
            "allowPartial": false
        },
        "expiration": "2019-05-18T13:54:49-05:00",
        "ipAddress": "190.85.82.130",
        "userAgent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 
                      (KHTML, like Gecko) Chrome/74.0.3729.131 Safari/537.36",
        "returnUrl": "https://dnetix.co/p2p/client",
        "cancelUrl": "https://dnetix.co",
        "skipResult": false,
        "noBuyerFill": false,
        "captureAddress": false,
        "paymentMethod": null,
        "fields": [
            {
            "keyword": "Redeem Code",
            "value": 754100,
            "displayOn": "payment"
            }
        ],
    ...
```
  
   Estructura que contiene la información acerca del pago de la transacción requerida al servicio web. <br><br>

ATRIBUTOS

Nombre | Tipo | Descripción
-------|------|------------
reference <code>Requerido</code> | string(32) | Única referencia para la solicitud de pago.
description <code>Opcional</code> | string | Descripción de la cuenta.
amount <code>Requerido</code> | <a href="#amount">Amount</a> | Objeto que contiene el monto a ser cobrado.
allowPartial <code>Requerido</code> | bool | Define si el monto a ser cobrado puede ser parcial.
shipping <code>Opcional</code> | [Person](#person) | Información de la persona quien recibe el producto o servicio en la transacción.
items <code>Opcional</code> | [Items](#items) | Productos relacionados con esta solicitud de pago.
fields <code>Opcional</code> | [NameValuePair[]](#namevaluepair) | Información adicional relacionada con la solicitud de pago que el comercio requiere guardar con la transacción.
recurring <code>Opcional</code> | [Recurring](#recurring) | Información recurrente cuando PlacetoPay procesa un pago recurrente.
subscribe <code>Opcional</code> | bool | Permite solicitar un proceso de cobro y suscripción en la misma sesión.
 
## SubscriptionRequest
   Estructura que contiene la información relacionada con una solicitud de suscripción para obtener un Token. <br><br>

ATRIBUTOS

Nombre | Tipo | Descripción
-------|------|------------
reference <code>Requerido</code> | string | Referencia única para la solicitud de suscripción
description <code>Opcional</code> | string | Descripción de la suscripción.
fields <code>Opcional</code> | [NameValuePair[]](#namevaluepair) | Información adicional relacionada con la suscripción.

**Retorna:** [SubscriptionResponse](#subscriptionresponse)

## SubscriptionResponse

>Ejemplo que contiene la estructura SubscriptionResponse:

  ```shell
  {
     ...
        "subscription": {
                "type": "token",
                "status": {
                    "status": "OK",
                    "reason": "00",
                    "message": "Token generado exitosamente",
                    "date": "2019-03-08T10:31:54-05:00"
                },
                "instrument": [
                    {
                        "keyword": "token",
                        "value": "1b394089c97bc5f155b684c068806d858d9e4180c8643b5b29cf216c4b656d3e",
                        "displayOn": "none"
                    },
     ...               
 }
  ```
Estructura que contiene información para el método de pago suscripción. <br><br>

ATRIBUTOS

Parametro | Tipo | Descripción
--------- | ---- | -----------
status <code>Requerido</code> | <a href="#status">Status</a>| Estado de la suscripción.
type <code>Opcional</code> | String | Esta cadena dicta el tipo de suscripción que se devuelve, puede ser [token, cuenta]
instrument <code>Opcional</code> | <a href="#namevaluepair">NameValuePair[]</a> | Acorde con el tipo de suscripción los valore retornados puede cambiar y serán devueltos en la estructura de NameValuePair.<br>**token:**<code>[token, subtoken, franchise, <br>franchiseName, lastDigits, <br> validUntil]</code><br>**account:** <code>[bankCode, bankName, accountType, <br>accountNumber]</code>

## NameValuePair

 >Ejemplo que contiene la estructura namevaluepair:

  ```shell
     ...
            "processorFields": [
                {
                    "keyword": "merchantCode",
                    "value": "011271442",
                    "displayOn": "none"
                },
                {
                    "keyword": "terminalNumber",
                    "value": "00057742",
                    "displayOn": "none"
                },
                {
                    "keyword": "lastDigits",
                    "value": "1111",
                    "displayOn": "none"
                },
                {
                    "keyword": "id",
                    "value": "42337fd0ed2b037667ccabce3427f042",
                    "displayOn": "none"
                },
                {
                    "keyword": "bin",
                    "value": "411111",
                    "displayOn": "none"
                },
                {
                    "keyword": "installments",
                    "value": "1",
                    "displayOn": "none"
                },
                {
                    "keyword": "expiration",
                    "value": "0320",
                    "displayOn": "none"
                },
                {
                    "keyword": "cardType",
                    "value": "C",
                    "displayOn": "none"
                }
            ]
    ...       
```
   Se utiliza para definir un tipo de par <code>clave-valor</code> <br><br>

ATRIBUTOS

Nombre | Tipo | Descripción
-------|------|------------
keyword <code>Opcional</code> | string | Clave para el par de valores del dato.
value <code>Opcional</code> | string |Valor para el par de datos.
displayOn <code>Opcional</code> | string | Bajo qué circunstancias el campo debe ser mostrado en la interfaz de redirección. [none, payment, receipt, both, approved]

## Status

>Ejemplo de una estructura status con respuesta fallida en una petición de autenticación:

```shell
{
    "status": {
        "status": "FAILED",
        "reason": 401,
        "message": "Authentication Failed 103",
        "date": "2019-03-05T10:22:11-05:00"
    }
}
```
   Estructura que contiene la información de la respuesta sobre una solicitud o pago, e informa el estado actual de la misma. <br><br>

ATRIBUTOS

Parametro | Tipo | Descripción
--------- | ---- | -----------
status <code>Requerido</code> | String(32) | <a href="#estado-de-una-peticion-o-pago">Estado de una petición o pago</a>
reason <code>Requerido</code> | String | Código del motivo proporcionado.
message <code>Requerido</code> | String | Descripción del código de razón.
date <code>Requerido</code> | DateTime | Fecha y hora en que se genera el estado de pago.

### Estado de una petición o pago

>Ejemplo de un estado de una petición o pago:

```shell
{
    "status": {
        "status": "REJECTED",
        "message": "",
        "reason": "",
        "date": "2016-09-15T13:49:01-05:00"
    },
    "requestId": 58,
    "reference": "ORDER-1000",
    "signature": "feb3e7cc76939c346f9640573a208662f30704ab"
}
```

Estado | Descripción
--------- | -----------
<code>OK</code> | Petición de autenticación procesada correctamente.
<code>FAILED</code> | Fallo en una petición de autenticación.
<code>APPROVED</code> | Petición de pago o suscripción terminada
<code>APPROVED_PARTIAL</code> | Si se permite parcialmente, se ha aprobado una transacción, pero aún queda un monto.
<code>PARTIAL_EXPIRED</code> | Cuando hay un pago parcial aprobado, pero la sesión ha caducado. 
<code>REJECTED</code> | Se ha declinado la transacción.
<code>PENDING</code> | Transacción pendiente para la sesión, debe estar bloqueada hasta la resolución.
<code>PENDING_VALIDATION</code> | La sesión está pendiente de validación de identidad del usuario.
<code>REFUNDED</code> | Reintegro de una transacción por solicitud de un tarjeta habiente al comercio.

## Transaction
   Estructura que contiene información sobre el proceso de pago de la transacción en PlacetoPay. <br><br>

ATRIBUTOS

Nombrre | Tipo | Descripción
--------|------|------------
status <code>Requerido</code> | [Status](#status) | Estado de la transacción.
internalReference <code>Opcional</code> | int | Referencia interna en PlacetoPay.
reference <code>Opcional</code> | string | Referencia enviada por el comercio para la transacción.
paymentMethod <code>Opcional</code> | string | Código del método de pago utilizado.
paymentMethodName <code>Opcional</code> | string | Nombre del método de pago utilizado.
issuerName <code>Opcional</code> | string | Nombre del emisor o del procesador.
amount <code>Opcional</code> | [AmountConversion](#amountconversion) | Valor procesado.
receipt <code>Opcional</code> | string | Numero de recibo de la transacción.
franchise <code>Opcional</code> | string | Franquicia de la tarjeta utilizada.
refunded <code>Opcional</code> | boolean |
authorization <code>Opcional</code> | string | Código de autorización.
processorFields <code>Opcional</code> | [NameValuePair[]](#namevaluepair) | Campos adicionales del procesador.

## Token
   Estructura que contiene información acerca de un token usado para cobros de un cliente suscrito. <br><br>

ATRIBUTOS

Nombre | Tipo | Descripción
-------|------|-------------
status <code>Requerido</code> | <a href="#status">Status</a> | Estado del proceso de tokenización.
token <code>Opcional</code> | string | Token completo para tarjeta de crédito, debe ser usada para solicitar cualquier transacción a PlacetoPay.
subtoken <code>Opcional</code> | string | Representación numérica del token para casos donde es requerido un número adicional que parece como una tarjeta de crédito, los últimos 4 dígitos son iguales a los últimos 4 dígitos de la tarjeta de crédito.
franchiseName <code>Opcional</code> | string | Franquicia de la tarjeta tokenizada.
issuerName <code>Opcional</code> | string | Nombre del banco emisor.
lastDigits <code>Opcional</code> | string | Últimos 4 dígitos de la tarjeta de crédito.
validUntil <code>Opcional</code> | date | Fecha hasta la cual el token es válido, puede  ser determinada por la  fecha de expiración.

## DocumentType
   Contiene los diferentes tipos de documento, cadena (string) (ver listado)

## Address
   Estructura que contiene la información sobre una dirección física. <br><br>

ATRIBUTOS

Nombre | Tipo | Descripción
-------|------|------------
street <code>Requerido</code> | string | Dirección física completa.
city <code>Requerido</code> | string | Nombre de la ciudad.
state <code>Opcional</code> | string | Nombre del estado coincidente con la dirección.
postalCode <code>Opcional</code> | string | Código postal o equivalente se requiere generalmente para los países que tienen.
country <code>Opcional</code> | string | Código internacional del país que se aplica a la dirección física según  ISO 3166-1 ALPHA-2.
phone <code>Opcional</code> | [PhoneNumberType](#phonenumbertype) | Número telefónico.

## PhoneNumberType
   Restringe la longitud del número de teléfono de la cadena a 30 caracteres.

## Amount

>Ejemplo de una esrtructura amount para un pago:


```shell
{
  ...
         "amount": {
           "taxes": [
                {
                "kind": "ice",
                "amount": 1.56,
                "base": 13
                },
                {
                "kind": "valueAddedTax",
                "amount": 2.47,
                "base": 13
                }
            ],
            "details": [
                {
                "kind": "shipping",
                "amount": 0.65
                },
                {
                "kind": "tip",
                "amount": 0.65
                },
                {
                "kind": "subtotal",
                "amount": 13
                }
            ],
            "currency": "COP",
            "total": "10000"
        }
  ...     
}
```

   Extiende de AmountBase y describe el contenido del valor total, incluyendo impuestos y detalles. <br><br>
   
ATRIBUTOS

Parametro | Tipo | Descripción
--------- | ---- | -----------
currency <code>Requerido</code> | String | De amountBase. Permite identificar la moneda en que se va realizar un pago, acorde al [ISO 4217](https://www.currency-iso.org/dam/downloads/lists/list_one.xml) (Alphabetic).
total <code>Requerido</code> | Decimal | De amountBase. Permite identificar el monto total que se va a pagar.
taxes <code>Opcional</code> | [TaxDetail[]](#taxdetail) | Descrpción de los impuestos.
details <code>Opcional</code> | [AmountDetail[]](#amountdetail) | Descripción del importe total.
## TaxDetail
  Estructura para almacenar información sobre un impuesto. <br><br>

Nombre | Tipo | Descripción 
-------|------|-------------
kind <code>Requerido</code> | string | Valor de clasificación, puede ser <code>[valueAddedTax, exciseDuty]</code>.
amount <code>Requerido</code> | [AmountType](#amounttype) | Valor discriminado.

## AmountDetail
   Estructura para almacenar información sobre el valor. <br><br>

ATRIBUTOS

Nombre | Tipo | Descripción
-------|------|------------
kind <code>Requerido</code> | string | Valor de clasificación, puede ser <code>[discount, additional, <br> vatDevolutionBase, shipping, <br> handlingFee, insurance, <br> giftWrap, subtotal, fee, tip]</code>.
amount <code>Requerido</code> | [AmountType](#amounttype) | Valor discriminado.

## AmountType
Representación decimal del valor.

## Recurring
Estructura que contiene la información requerida para una solicitud de pago recurrente. <br><br>

ATRIBUTOS

Nombre | Tipo | Descripción
-------|------|-------------
periodicity <code>Requerido</code> | string | Periodicidad de la factura <code>[D, M, Y]</code>
interval <code>Requerido</code> | int | Intervalo asociado a la periodicidad.
nextPayment <code>Requerido</code> | date | Fecha del próximo pago.
maxPeriods <code>Requerido</code> | int | Número máximo de periodo (-1 en caso de que no haya  restricción.)
dueDate <code>Opcional</code> | date | Fecha para finalizar el pago.
notificationUrl <code>Opcional</code> | string | URL en el que el servicio notificará cada vez que se haga un pago recurrente.

## AmountConversion
   Estructura para definir el factor de conversión y los valores. <br><br>

ATRIBUTOS

Nombre | Tipo | Descripción
-------|------|--------------
from <code>Requerido</code> | [AmountBase](#amountbase) | Monto solicitado.
to <code>Requerido</code> | [AmountBase](#amountbase) | Monto procesado por la entidad.
factor <code>Requerido</code> | double | Factor de conversión.


## AmountBase
   Estructura que representa una cantidad que define la moneda y el total. <br><br>

ATRIBUTOS

Nombre | Tipo | Descripción
-------|------|------------
currency <code>Requerido</code> | string | Moneda acorde al ISO 4217 <code>(alphabetic code)</code>.
total <code>Requerido</code> | decimal | Valor total.

## CollectRequest

>Ejemplo para recaudar un pago utilizando la estructura CollecRequest

```shell
POST /api/collect
{
    "auth": {
	    "login": "usuarioprueba",
	    "tranKey": "YhmdBEZKTMAFsxkI1SrgS4SdBfE=",
	    "nonce": "TlRoaE0yWTROMk0wWVRFellXUTVNVFUwWVRZNVpHRXpZbU16TWpJM01UWT0=",
	    "seed": "2019-04-25T18:17:23-04:00"
  },
    "instrument": {
        "token": {
            "token": "1b394089c97bc5f155b684c068806d858d9e4180c8643b5b29cf216c4b656d3e"
        }
    },
    "payer": {
            "document": "911111111",
            "documentType": "CC",
            "name": "Prueba aliado",
            "surname": "JA",
            "email": "juan.jimenez@placetopay.com"
    },
    "payment": {
        "reference": "3110",
        "description": "Pago con suscripción 3110",
        "amount": {
            "currency": "COP",
            "total":10000
        }
    }
}
```
Permite realizar cobros sin la intervención del usuario usando medios de pago previamente suscritos. <br><br>

ATRIBUTOS

Parametro | Tipo | Descripción
--------- | ---- | -----------
payer <code>Requerido</code> | <a href="#person">Person</a> | Datos del titular del medio de pago almacenado.
payment <code>Requerido</code> | <a href="#paymentrequest">PaymentRequest</a> | Objeto de pago cuando necesite solicitar un cobro.
instrument <code>Requerido</code> | <a href="#instrument">Instrument</a> | Datos asociados al medio de pago suscrito.

## Items
   Posee una colección de estructuras de elementos. <br><br>

ATRIBUTOS

Nombre | Tipo | Descripción
-------|------|------------
item <code>Opcional</code> | [Item[]](#item) | Arreglo de un elemento incluido.

## Item 
   Estructura que contiene los detalles del elemento. <br><br>
   
ATRIBUTOS

Nombre | Tipo | Descripción
-------|------|------------
sku <code>Requerido</code> | string | Unidad en stock correspondiente (SKU) al artículo.
name <code>Opcional</code> | string | Nombre del artículo.
category <code>Opcional</code> | string | Puede ser [digital, physical].
qty <code>Opcional</code> | string | Número de un artículo en particular.
price <code>Opcional</code> | decimal | Costo del artículo.
tax <code>Opcional</code> | decimal | Impuesto del artículo.

## Instrument

>Ejemplo que contiene una estructura instrument:

  ```shell
   {
     ...
        "instrument": {
           "token": {
               "token": "1b394089c97bc5f155b684c068806d858d9e4180c8643b5b29cf216c4b656d3e"
            }
       },
     ...               
    }
  ```
   Estructura que contiene los detalles de un medio de pago suscrito. <br><br>

ATRIBUTO

Nombre | Tipo | Descripción
-------|------|------------
token <code>Requerido</code> | [SimpleToken](#simpletoken) | Datos asociados a una tarjeta de crédito tokenizada.

## SimpleToken 
  Estructura que contiene los detalles de un token previamente obtenido mediante un proceso de suscripción, se debe enviar el token o el subtoken en los casos que se habilite, no es necesario enviar ambos. <br><br>

ATRIBUTOS

Nombre | Tipo | Descripción
-------|------|------------
token <code>Opcional</code> |string | Token completo para tarjeta de crédito, debe ser usada para solicitar cualquier transacción a PlacetoPay.
subtoken <code>Opcional</code> | string | Representación numérica del Token para casos donde es requerido un número adicional que parece como una tarjeta de crédito, los últimos 4 dígitos son iguales a los últimos 4 dígitos de la tarjeta de crédito.
installments <code>Opcional</code> | int | Número de cuotas en las cuales se solicita el cobro (opcional).
cvv <code>Opcional</code> | string | Dígitos del código de seguridad de la tarjeta a usar en los casos en los que sea necesario, generalmente se deja en blanco si se tiene una terminal sin validación de CVV.