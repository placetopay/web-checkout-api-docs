---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>
  - <a href='https://github.com/lord/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true
---

# Web Checkout


 Las especificaciones del servicio web y su arquitectura permite a cualquier aplicación conectarse a PlacetoPay. 

 Para hacer uso del servicio web es indispensable un lenguaje de programación que pueda conectarse con servicios REST y se conecte usando la versión 1.2 de TLS.

 En esta API se describen, los parámetros de ingreso y estructuras de datos utilizadas para conectarse con el web service <code>Web Checkout</code>.

**Consideraciones:**

* Se necesita la URL donde se van a enviar los datos mediante el uso del método POST (Endpoint).

* Se debe usar JSON para el envío de los parámetros de Autenticación, Información del pago y Datos de control.

* El tiempo de expiración <code>expiration</code> debe ser una fecha válida en formato ISO 8601, indica el tiempo
límite que tiene el usuario para realizar el pago y debe ser de al menos 5 minutos superior a la fecha actual.

**Importante:** El <code>expiration</code> es el tiempo máximo que se da para la sesión, si un usuario no realiza la operación la sesión queda pendiente hasta expirar. 

* La url de retorno (returnUrl) es utilizada para redirigir al usuario una vez terminada la operación. Se recomienda enviar en la URL un dato con el cual el sitio pueda identificar el registro por ejemplo: <code>https://mysite.com/response/{reference}</code>.

* Configura en tu servidor una URL de notificación usando los puertos 80 ó 443 y en caso de que el pago se decline o apruebe, reciba una notificación mediante POST de forma asincrona.

**Importante:** Esta URL de notificación debe ser suministrada a PlacetoPay para su configuración.

# Autenticación

```shell
Ejemplo:
    Información proporcionada:
    login = usuarioprueba
    secretKey = ABCD1234

    Datos generados por el usuario:
    nonce = c9085e82debb82b0955579098be3d7ca //Nonce original
    seed = 2019-04-25T18:17:23-04:00

    Datos a enviar:
    tranKey = i/RFwSHAh8d7YgtO3HME5kCnYy8=
    nonce = YzkwODVlODJkZWJiODJiMDk1NTU3OTA5OGJlM2Q3Y2E= //Nonce en base64
```

```shell
Ejemplo:
POST /api/session

{
  "auth": {
    "login": "usuarioprueba",
    "tranKey": "T0O+x3gNlQUf0iBxEuenPvBPlWs=",
    "nonce": "YzkwODVlODJkZWJiODJiMDk1NTU3OTA5OGJlM2Q3Y2E=",
    "seed": "2019-04-25T18:17:23-04:00",
  }
}
```

La autenticación al servicio debe ser enviada sobre el objeto <code>auth</code>, el cual debe contener los siguientes atributos:

Parametro | Descripción
--------- | -----------
login | Identificador del sitio.
tranKey | Llave transaccional, que está formada por la siguiente operación: Base64(SHA-1(nonce + seed + secretkey)). El nonce dentro de la operación es el original, es decir, el que no está en Base64.
nonce | Valor aleatorio para cada solicitud codificado en Base64.
seed | Fecha actual, la cual se genera en formato ISO 8601.

<aside class="warning">
El <code>seed</code> no puede tener diferencia de más de 5 minutos.
</aside>

<aside class="notice">
El login y el secretKey son suministrados por PlacetoPay 
</aside>

<aside class="notice">
Esta estructura debe de ser enviada en cada petición realizada al servicio 
</aside>

# Pago básico

>Ejemplo para la petición de un pago básico:

```shell
POST /api/session

{
  "auth": {
    "login": "usuarioprueba",
    "tranKey": "jsHJzM3+XG754wXh+aBvi70D9/4=",
    "nonce": "TTJSa05UVmtNR000TlRrM1pqQTRNV1EREprWkRVMU9EZz0=",
    "seed": "2019-04-25T18:17:23-04:00"
  },
 
    "payment": {
        "reference": "3210",
        "description": "Pago básico de prueba 04032019",
        "amount": {
            "currency": "COP",
            "total": "10000"
        }
      },

    "expiration": "2019-03-05T00:00:00-05:00",
    "returnUrl": "https://mysite.com/response/3210",
    "ipAddress": "127.0.0.1",
    "userAgent": "PlacetoPay Sandbox"
}
```

Cuando el usuario selecciona pagar en tu sitio, inicia el flujo del servicio Web Checkout, donde su sitio envía al EndPoint de procesamiento <code>(Ej: https://test.placetopay.com/redirection/api/session/)</code>, mediante el método POST los parametros necesarios para crear la sesión de pago.

En la estructura se deben tener en cuenta algunos parámetros que se manejan dependiendo del tipo de transacción (<a href="#pago-basico">Pago básico</a>, <a href="#pago-mixto">mixto</a>, o <a href="#pago-recurrente">recurrente</a>):

La siguiente información es necesaria para consumir el <code>CreateRequest</code>:

## Payment

>Ejemplo de una estructura payment:

```shell
{
...
    "payment": {
        "reference": "3210",
        "description": "Pago básico de prueba 04032019",
        "amount": {
            "currency": "COP",
            "total": "10000"
        }
      },
...
}
```

Estructura que contiene la información acerca del pago de la transacción requerida al servicio web.

Parametro | Tipo | Descripción
--------- | ---- | -----------
reference | String(32) | Esta debe ser única en la solicitud de un pago.
description | String | Permite identificar la descripción del pago.
amount | <a href="#amount">Amount</a> | Objeto que contiene el monto a ser cobrado. 


## Amount<br>

>Ejemplo de una esrtructura amount para un pago básico:

```shell
{
  ...
        "amount": {
            "currency": "COP",
            "total": "10000"
        }
  ...     
}
```

Contiene diferentes atributos que determinan el contenido del valor total, incluyendo los impuestos y detalles. 

Parametro | Tipo | Descripción
--------- | ---- | -----------
currency | String | Permite identificar la moneda en que se va realizar un pago, acorde al [ISO 4217](https://www.currency-iso.org/dam/downloads/lists/list_one.xml) (Alphabetic).
total | Decimal | Permite identificar el monto total que se va a pagar.
taxes | TaxDetail[] | Descrpción de los impuestos.
details | AmountDetail[] | Descripción del importe total.

Atributos que permiten generar una petición de redirección:

Parametro | Tipo | Descripción
--------- | ---- | -----------
expiration | DateTime | Fecha de expiración de una solicitud, el usuario debe terminar el proceso antes de esta fecha. i.e. 2019-03-05T00:00:00-05:00.
returnurl | String | URL de retorno es utilizada para redirigir al usuario una vez termina la operación de pago.
ipAddress | String | Dirección IP del usuario.
userAgent | String | Agente de usuario informado por el usuario.

<aside class="notice">
En el returnUrl se recomienda enviar en la URL un dato con el cual el sitio pueda identificar el registro por ejemplo: <code>https://mysite.com/response/{reference}</code>.
</aside>

## RedirectResponse

>Ejemplo de una estructura RedirectResponse con respuesta aprobada en una solicitud de autenticación:

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

Estructura que contiene la petición de respuesta inicial desde el método <code>createRequest.</code>

Parametro | Tipo | Descripción
--------- | ---- | -----------
status | <a href="#status">Status</a> | Estructura que contiene el estado de la solicitud de pago. 
requestId | Int | Referencia única de la sesión de pago.
processUrl | String | URL donde se redirecciona al usuario para completar el proceso de pago.


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

Estructura que contiene la información de la respuesta sobre una solicitud o pago, e informa el estado actual de la misma.

Parametro | Tipo | Descripción
--------- | ---- | -----------
status | String(32) | <a href="#estado-de-la-solicitud-o-pago">Estado de la solicitud o pago</a>
reason | String | Código del motivo proporcionado.
message | String | Descripción del código de razón.
date | DateTime | Fecha y hora en que se genera el estado de pago.

## Estado de la solicitud o pago

>Ejemplo de un estado de solicitud de pago con respuesta aprobada en una solicitud de autenticación:

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

Estado | Descripción
--------- | -----------
<code>OK</code> | Peticiíon de autenticación procesada correctamente.
<code>FAILED</code> | Fallo en una petición de autenticación.
<code>APPROVED</code> | Petición de pago o suscripción terminada
<code>APPROVED_PARTIAL</code> | Si se permite parcialmente, se ha aprobado una transacción, pero aún queda un monto.
<code>PARTIAL_EXPIRED</code> | Cuando hay un pago parcial aprobado, pero la sesión ha caducado. 
<code>REJECTED</code> | Se ha declinado la transacción.
<code>PENDING</code> | Transacción pendiente para la sesión, debe estar bloqueada hasta la resolución.
<code>PENDING_VALIDATION</code> | La sesión está pendiente de validación de identidad del usuario.
<code>REFUNDED</code> | Reintegro de una transacción por solicitud de un tarjeta habiente al comercio.

* Al recibir la respuesta del <a href="#redirectresponse">redirectResponse</a>, esta debe ser almacenada en su sistema. La operación debe quedar marcada como pendiente. <br>
* Luego de almacenar la respuesta de la operación, el sitio debe redireccionar al usuario sobre la misma pestaña a la interfaz de la sesión en Web Checkout (processUrl) donde debe ingresar la información para procesar el pago:


Parametro | Tipo | Obligatoriedad
--------- | ---- | -----------
correo electrónico | String | Si
tipo de documento |  DocumentType | Si
número de documento | String | Si
nombre | String | Si
apellidos | String | Si
celular | String | Si

* Posteriormente el usuario debe seleccionar el medio de pago con el que va realizar la compra.
* Finalmente el usuario ingresa los datos del medio de pago:

Parametro | Tipo | Obligatoriedad
--------- | ---- | -----------
número de tarjeta | String | Si
fecha de vencimiento | dateTime | Si
código de seguridad | String | Si
cuotas | String | Si

* El servicio Web Checkout procesa la operación y retorna a su sistema <code>(returnUrl)</code>. 
* Su sistema consulta la información de la sesión.
* Su sistema almacena los datos necesarios obtenidos en la respuesta de la sesión.
* Su sistema presenta al usuario la información de la operación.


<aside class="notice">
En caso que la operación quede pendiente, el servicio notificará de forma asíncrona a la URL de notificación una vez sea resuelta (Apruebe o decline)
</aside>


# Pago mixto

>Ejemplo de una petición para un pago mixto.

```shell
POST /api/session

{
  "auth": {
    "login": "usuarioprueba",
    "tranKey": "jsHJzM3+XG754wXh+aBvi70D9/4=",
    "nonce": "TTJSa05UVmtNR000TlRrM1pqQTRNV1EREprWkRVMU9EZz0=",
    "seed": "2019-04-25T18:17:23-04:00"
  },
 
   "payment": {
        "reference": "3210",
        "description": "Pago mixto de prueba",
        "amount": {
            "currency": "COP",
            "total": "10000"
        },
        "allowPartial": true
    },

    "expiration": "2019-03-05T00:00:00-05:00",
    "returnUrl": "https://mysite.com/response/3210",
    "ipAddress": "127.0.0.1",
    "userAgent": "PlacetoPay Sandbox"
}
```

>Ejemplo aprobación  de una transacción para un pago mixto:

```shell
{
    ...
    "status": {
        "status": "APPROVED",
        "reason": "P0",
        "message": "La petición está parcialmente aprobada",
        "date": "2019-04-01T11:13:27-05:00"
    },
    ...
}
```

>Ejemplo aprobación parcial de una transacción para un pago mixto:

```shell
{
    ...
    "status": {
        "status": "APPROVED_PARTIAL",
        "reason": "P0",
        "message": "La petición está parcialmente aprobada",
        "date": "2019-04-01T11:13:27-05:00"
    },
    ...
}
```

>Ejemplo de un pago mixto con saldo pendiente y el tiempo de expiración finalizado:

```shell
{
    ...
    "status": {
        "status": "PARTIAL_EXPIRED",
        "reason": "PX",
        "message": "La petición esta expirada o cancelada y se han realizado pagos",
        "date": "2019-04-01T11:15:38-05:00"
    },
    ...
}
```

>Ejemplo de un pago mixto en estado pendiente:

```shell
{
    ...
    "status": {
        "status": "PENDING",
        "reason": "PC",
        "message": "La petición se encuentra activa",
        "date": "2019-04-25T11:25:53-05:00"
    },
    ...
}
```

El pago mixto permite utilizar varios medios de pago al momento de pagar para una misma sesión.

Para hacer uso de esta funcionalidad, en la estructura <code>payment</code> del <a href="#pago-basico">Pago básico</a> es necesario 
enviar el atributo <code>allowPartial</code> como: <code>TRUE</code>.


Parametro | Tipo | Descripción
--------- | ---- | -----------
allowPartial | Bool | Define si la operación permite pago mixto.

**Estados adicionales de una sesión de pago mixto**

Al obtener el resultado de la operación, debe tener muy presente los siguiente dos estados:

* <code>APPROVED_PARTIAL:</code> Indica que se realiza al menos un pago aprobado pero no se realizado el pago total de la sesión.
* <code>PARTIAL_EXPIRED:</code> Indica que se realiza al menos un pago aprobado pero no se completó el pago total de la sesión y esta expiró.

<aside class="notice">
 Es importante identificar según la regla de negocio del sitio al obtener alguno de estos dos estados.
</aside>

# Pago Recurrente

>Ejemplo petición para un pago recurrente:

```shell
POST /api/session

{
  "auth": {
    "login": "usuarioprueba",
    "tranKey": "jsHJzM3+XG754wXh+aBvi70D9/4=",
    "nonce": "TTJSa05UVmtNR000TlRrM1pqQTRNV1EREprWkRVMU9EZz0=",
    "seed": "2019-04-25T18:17:23-04:00"
    },
    "payment": {
        "reference": "3210",
        "description": "Pago recurrente de prueba",
        "amount": {
            "currency": "COP",
            "total": "10000"
        },
        "recurring": {
            "periodicity": "M",
            "interval": "1",
            "nextPayment": "2019-04-04",
            "maxPeriods": "12"
        }
    },
    "expiration": "2019-03-05T00:00:00-05:00",
    "returnUrl": "https://mysite.com/response/3210",
    "ipAddress": "127.0.0.1",
    "userAgent": "PlacetoPay Sandbox"
}
```

Es un cobro periódico realizado por PlacetoPay para un  **mismo valor** con un intervalo (diario, mensual, anual) según la indicación dada en la petición.

Para hacer uso de esta funcionalidad, en la estructura <code>payment</code> del Pago básico es necesario enviar en el objeto <code> recurring</code> la siguiente información:

Parametro | Tipo | Descripción
--------- | ---- | -----------
periodicity | String | Define periodicidad de un  pago.  [D, M, Y].
interval | Int | Número de Días, Meses o Años asociado a la periodicidad (-1 representa un intervalo indefinido).
nexPayment | Date | Fecha del próximo pago con formato AAAA-MM-DD.
maxPeriods | Int | Número máximo de periodos a recaudar (-1 en caso de que no haya  restricción.)

<aside class="notice">
Los cobros recurrentes sólo se realizarán en el caso que  la transacción inicial sea aprobada.
</aside>

<aside class="notice">
En el caso de fallar un cobro recurrente, este será reintentado cada día durante N días, si luego del número máximo de reintentos no se obtiene una transacción aprobada, la reccurencia se cancela.
</aside>

<aside class="notice">
Las recurrencias sólo pueden ser canceladas en la consola administrativa de PlacetoPay.
</aside>

# Suscripción

>Ejemplo petición para una suscripción:

```shell
{
   "auth":| {
    "login": "usuarioprueba",
    "tranKey": "jsHJzM3+XG754wXh+aBvi70D9/4=",
    "nonce": "TTJSa05UVmtNR000TlRrM1pqQTRNV1EREprWkRVMU9EZz0=",
    "seed": "2019-04-25T18:17:23-04:00"
    },
    "subscription": {
        "reference": "3110",
        "description": "Una suscripción de prueba"
    },
    "expiration": "2019-08-02T00:00:00-05:00",
    "returnUrl": "https://mysite.com/response/3210",
    "ipAddress": "127.0.0.1",
    "userAgent": "PlacetoPay Sandbox"
}
```
La suscripción permite almacenar de forma segura (tokenizando) el medio de pago de un usuario, para que luego pueda efectuar cobros relacionados a este mismo.

Para hacer uso de esta funcionalidad es necesario reemplazar en la solicitud al servicio la  estructura <code>Payment</code> por <code>Subscription</code> en la cual deben ser enviada la siguiente información:

Parametro | Tipo | Descripción
--------- | ---- | -----------
reference | String | Esta debe ser única en la solicitud de suscripción.
description | String | Descripción de la suscripción

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


Estructura que contiene información para el método de pago con suscripción.

Parametro | Tipo | Descripción
--------- | ---- | -----------
status | Status | Estado de la suscripción.
type | String | Esta cadena dicta el tipo de suscripción que se devuelve, puede ser [token, cuenta]
instrument | <a href="#namevaluepair">NameValuePair[]</a> | Acorde con el tipo de suscripción los valore retornados puede cambiar y serán devueltos en la estructura de NameValuePair.<br>**token:**<code>[token, subtoken, franchise, <br>franchiseName, lastDigits, validUntil]</code><br>**account:** <code>[bankCode, bankName, accountType, <br>accountNumber]</code> 

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

Se utiliza para definir un tipo de par <code>clave-valor</code>

Parametro | Tipo | Descripción
--------- | ---- | -----------
keyword | String | Clave para el par de valores del dato.
value | String | Valor para el par de datos.
displayOn | String | Bajo qué circunstancias el campo debe ser mostrado en la interfaz de redirección. <br><code>[none, payment, receipt, both, approved]</code>


<aside class="notice">
Se recomienda realizar un cobro por un valor mínimo para identificar que la tarjeta se encuentra activa.
</aside>

<aside class="notice">
La información del token debe ser almacenada en su sitio y relacionada al usuario y/o producto.
Luego de tener el token, puede generar cobros al medio de pago tokenizado usando el servicio <a href="#collect">collect</a>.
</aside>

La estructura de la respuesta contiene toda la información de la petición original y una estructura de suscripción <code>(subscription)</code> con un instrumento <code>(instrument)</code> representado en forma de **token**. [Ver Ejemplo de respuesta de una suscripción](#consulta).

## Recaudar pago suscripción 

Con el <code>token</code> obtenido se procede a realizar la petición de recaudo del pago a la URL de procesamiento <code>(Ej: https://test.placetopay.com/redirection/api/collect/)</code> sin intervención del usuario, mediante el método POST y los parametros con estructuras JSON. Adicionalmente debe enviarse una estructura de pago <code>(payment)</code> y pagador <code>(payer)</code>.

### Collect

>Ejemplo para recaudar un pago con suscripción

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
Permite realizar cobros sin la intervención del usuario usando medios de pago previamente suscritos.

Parametro | Tipo | Descripción
--------- | ---- | -----------
payer | <a href="#person">Person</a> | Datos del titular del medio de pago almacenado.
payment | <a href="#payment">PaymentRequest</a> | Objeto de pago cuando necesite solicitar un cobro.
instrument | <a href="#instrument">Instrument</a> | Datos asociados al medio de pago suscrito.

###Person
Estructura que refleja la información de una persona involucrada en una transacción.

Parametro | Tipo | Descripción
--------- | ---- | -----------
documentType | DocumentType | <a href="#tipos-de-documento">Tipo de documento</a>
document | String | Identificación.
name | String | Nombres de la persona.
surname | String | Apellidos de la persona.
company | String | Nombre de la empresa en la que trabaja o representa.
email | String | Correo electrónico de la persona.
address | Address | Información completa de la dirección.
mobile | PhoneNumberType | Número celular, el <code>PhoneNumberType</code> restringe la longitud del número de teléfono de la cadena a 30 caracteres.

## Tipos de documento
Esta es una lista de los tipos de docuemntos aceptados por redirección.

País | Código | Tipo de documento
--------- | ---- | -----------
Internacional | PPN | Pasaporte
              | TAX | TAX
              | LIC | Labeler Identification Code
Colombia | CC | Cédula de ciudadanía
         | CE | Cédula de extranjería
         | TI | Tarjeta de identidad
         | RC | Registro Civil
         | NIT | Número de Identificación Tributaria
         | RUT | Registro único tributario
Ecuador | CI | Cédula de identidad
        | RUC | Registro Único de Contribuyentes
Panamá | CIP | Cédula de identidad personal
Brazil | CPF | Cadastro de Pessoas Físicas
USA | SSN | Social security number


**Pago recurrente vs Suscripción**<br>
  Las principales diferencias entre estas dos funcionalidades son:

  * El pago recurrente es ejecutado por PlacetoPay mientras que la suscripción es ejecutada por su sitio. 
  * En un pago recurrente ya creado no se puede modificar los intervalos de tiempo para el cobro ni el monto, mientras que en suscripción la información puede variar dependiendo de cómo envie la solicitud en el collect.
  * Para eliminar la recurrencia de un pago debe hacerse por la consola administrativa de PlacetoPay, mientras que en suscripción debe invalidar o eliminar de sus sistema el token obtenido.


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
Estructura que contiene los detalles de un medio de pago suscrito.

**Token**<br> 
Estructura que contiene información acerca de un token usado para cobros de un cliente suscrito.

Parametro | Tipo | Descripción
--------- | ---- | -----------
token | String | Token completo para tarjeta de crédito, debe ser usado para solicitar cualquier transacción a PlacetoPay.


**Payer**<br>

>Ejemplo que contiene una estructura payer:

```shell
   {
    ...
        "payer": {
                    "document": "98762729",
                    "documentType": "CE",
                    "name": "JD",
                    "surname": "JA",
                    "email": "juan.jimenez@placetopay.com",
                    "mobile": "321000000"
                },
    ...
   }
```

Estructura que contiene los datos del titular del medio de pago almacenado.

Parametro | Tipo | Descripción
--------- | ---- | -----------
document | String | Identificación del usuario.
documentType | <a href="#tipos-de-documento">DocumentType</a> | Tipo de identificación del usuario pagador.
name | String | Nombres del usuario pagador.
surname | String | Apellidos del usuario pagador.
email | String | Correo electronico del usuario pagador.
mobile | PhoneNumberType | Número telefónico.


<aside class="notice">
Si el usuario retorna al comercio, este debe consultar la información de la transacción usando el identificador único (requestId) generado.
</aside>

# Notificación

>Ejemplo de notificación para un pago aprobado:

```shell
{
    "status": {
        "status": "APPROVED",
        "message": "",
        "reason": "",
        "date": "2016-09-15T13:49:01-05:00"
    },
    "requestId": 58,
    "reference": "ORDER-1000",
    "signature": "feb3e7cc76939c346f9640573a208662f30704ab"
}
```

>Ejemplo de notificación para un pago pendiente:

```shell
{
    "status": {
        "status": "PENDING",
        "message": "",
        "reason": "",
        "date": "2016-09-15T13:49:01-05:00"
    },
    "requestId": 58,
    "reference": "ORDER-1000",
    "signature": "feb3e7cc76939c346f9640573a208662f30704ab"
}
```

>Ejemplo de notificación para un pago declinado:

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

Una notificación permite informar al comercio sobre el estado de las sesiones.

Configura en tu servidor una URL de notificación usando los puertos <code>80</code> ó <code>443</code> y en caso de que el pago se cancele o apruebe, reciba una notificación mediante POST.

Parametro | Tipo | Descripción
--------- | ---- | -----------
requestId | Int | Identificador unico de la sesión.
reference | String | Reeferencia que acompañada con el <code>requestId</code> permite identificar la sesión de pago afectada.
signature | String | Firma que permite validar la petición recibida.

De manera asincrónica, una notificación del estado de la petición será enviada a una URL definida por configuración del comercio en PlacetoPay.

Al recibir la  notificación su sitio debe:

**Identificar operación:** <br>
Con el identificador de la sesión <code>requestId</code> y la referencia <code>reference</code> identifican la sesión notificada.
Para esta sesión debe validar que el estado de la operación notificada están pendientes en su sistema.

**Validar autenticidad de la notificación:** <br>
Puedes validar que se trate de una respuesta de PlacetoPay haciendo un SHA-1 con los datos <code>requestId + status + date + secretKey</code>.

Si este valor coincide con el valor proporcionado por el <code>signature</code>, puedes autenticar la respuesta y proceder a asentar en la base de datos.


<aside class="warning">Bajo ninguna circunstancia reveles tu <code>secretKey</code> en ninguna parte accesible a los clientes o usuarios de su aplicación.
</aside>

<aside class="success">Luego de obtener el estado de la sesión, puede continuar con la regla de negocio de su sitio. (ejemplo: si la operación es aprobada, envia un correo electrónico al usuario).
</aside>

# Consulta

>Ejemplo de la petición de consulta de la información de una transacción:

```shell
POST /api/session/181348

{
  "auth": {
    "login": "usuarioprueba",
    "tranKey": "NBV2D/QzFUMQi3G/bvkie/HMjdM=",
    "nonce": "WXpVME1tTXlNR1l3TVRJM01XUTJNREJqWlRGaU9Ua3hPVE0wT1dReU9EQT0=",
    "seed": "2019-04-25T18:17:23-04:00"
  }
}
```

>Ejemplo respuesta consulta de un pago:

```shell
{
    "requestId": 181348,
    "status": {
        "status": "APPROVED",
        "reason": "00",
        "message": "La petición ha sido aprobada exitosamente",
        "date": "2019-03-04T17:27:59-05:00"
    },
    "request": {
        "locale": "es_CO",
        "payer": {
            "document": "98762729",
            "documentType": "CE",
            "name": "JD",
            "surname": "JA",
            "email": "juan.jimenez@placetopay.com",
            "mobile": "321000000"
        },
        "payment": {
            "reference": "3210",
            "description": "Pago básico de prueba 04/03/2019",
            "amount": {
                "currency": "COP",
                "total": "10000"
            },
            "allowPartial": false,
            "subscribe": false
        },
        "fields": [
            {
                "keyword": "_processUrl_",
                "value": "https://test.placetopay.com/redirection/session/181348/43d83d36aa46de5f993aafb9b3e0be48",
                "displayOn": "none"
            }
        ],
        "returnUrl": "https://mysite.com/response/3210",
        "ipAddress": "127.0.0.1",
        "userAgent": "PlacetoPay Sandbox",
        "expiration": "2019-03-05T00:00:00-05:00"
    },
    "payment": [
        {
            "status": {
                "status": "APPROVED",
                "reason": "00",
                "message": "Aprobada",
                "date": "2019-03-04T17:05:00-05:00"
            },
            "internalReference": 1468647381,
            "paymentMethod": "card",
            "paymentMethodName": "Visa",
            "issuerName": "BANCO DE PRUEBAS",
            "amount": {
                "from": {
                    "currency": "COP",
                    "total": "10000.00"
                },
                "to": {
                    "currency": "COP",
                    "total": "9800.00"
                },
                "factor": 1
            },
            "authorization": "000000",
            "reference": "3210",
            "receipt": "1551737100",
            "franchise": "CR_VS",
            "refunded": false,
            "discount": {
                "code": "DEMO_PROMOVISA",
                "type": "MERCHANT",
                "amount": "200",
                "base": "10000",
                "percent": 2
            },
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
        }
    ],
    "subscription": null
}
```

>Ejemplo de respuesta de una suscripción:

```shell
{
    "requestId": 183064,
    "status": {
        "status": "APPROVED",
        "reason": "00",
        "message": "La petición ha sido aprobada exitosamente",
        "date": "2019-03-08T10:33:09-05:00"
    },
    "request": {
        "locale": "es_CO",
        "payer": {
            "document": "911111111",
            "documentType": "CC",
            "name": "Prueba aliado",
            "surname": "JA",
            "email": "juan.jimenez@placetopay.com",
            "mobile": "321000000"
        },
        "subscription": {
            "reference": "3110",
            "description": "Una suscripción de prueba"
        },
        "returnUrl": "https://mysite.com/response/3110",
        "ipAddress": "127.0.0.1",
        "userAgent": "PlacetoPay Sandbox",
        "expiration": "2019-04-09T00:00:00-05:00"
    },
    "payment": null,
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
            {
                "keyword": "subtoken",
                "value": "2364106985941111",
                "displayOn": "none"
            },
            {
                "keyword": "franchise",
                "value": "visa",
                "displayOn": "none"
            },
            {
                "keyword": "franchiseName",
                "value": "Visa",
                "displayOn": "none"
            },
            {
                "keyword": "issuerName",
                "value": "JPMORGAN CHASE BANK, N.A.",
                "displayOn": "none"
            },
            {
                "keyword": "lastDigits",
                "value": "1111",
                "displayOn": "none"
            },
            {
                "keyword": "validUntil",
                "value": "2021-03-31",
                "displayOn": "none"
            },
            {
                "keyword": "installments",
                "value": "1",
                "displayOn": "none"
            }
        ]
    }
}
```


La consulta de información de una transacción se puede dar cuando el usuario es retornado a su sitio ó mediante una tarea programada que diariamente ejecute un script en el cual se consulten todas las operaciones pendientes en su sitio. 

**Recomendación:** Implementar en su sitio las dos alternativas, pues si algo falla en el proceso de retorno y notificación, la tarea programada puede ser usada como **contingencia**.

Para consultar la sesión debe enviar el identificador único de la sesión de pago <code>requesId</code>, mediante el método POST se envían los datos de autenticación y se genera la respuesta de la transacción (<code>endpoint:  https://test.placetopay.com/redirection/api/session/**{requestid}**</code>).

<aside class="notice">
Para este consumo también es necesario enviar en la petición la autenticación.
</aside>

La respuesta de la transacción debe contener diferentes objetos: 

* <a href="#status">status</a>

* **request**
Contiene 

Datos adicionales a la estructura <code>payment</code>

Parametro | Tipo | Descripción
--------- | ---- | -----------
allowPartial | Bool | Define si el monto a ser cobrado puede ser parcial.
suscribe | Bool | Permite solicitar un proceso de cobro y suscripción en la misma sesión.

**fields**<br>

Estructura de tipo <code>nameValuePair</code> que se utiliza para definir un tipo de par <code>clave-valor</code><br>.
Estructura que con la información adicional relacionada con la solicitud que el comercio requiere para guardar con la transacción.

Parametro | Tipo | Descripción
--------- | ---- | -----------
keyword | String | Clave para el par de valores del dato
value | String | Valor para el par de datos 
displayOn | String | Define bajo qué circunstancias el campo debe ser mostrado en la interfaz de redirección (none, payment, receipt, both, approved).

Datos de la transacción:

Parametro | Tipo | Descripción
--------- | ---- | -----------
internalReference | Int | Referencia interna en PlacetoPay.
paymentMethod | String | Código del método de pago utilizado.
paymentMethodName | String | Nombre del método de pago utilizado.
issuerName | String | Nombre del emisor o del procesador.