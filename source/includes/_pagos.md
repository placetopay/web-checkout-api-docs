# Pagos

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
 
    "payment": {
        "reference": "3210",
        "description": "Pago de prueba 04032019",
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

Cuando el usuario selecciona pagar en tu sitio, inicia el flujo del servicio Web Checkout, donde su sitio envía al EndPoint de procesamiento mediante el método POST los parámetros necesarios para crear la sesión de pago.

En la estructura se deben tener en cuenta algunos parámetros para consumir el método [CreateRequest](#createrequest) los cuales estan contenidos en la estructura de [Autenticación](#autenticacion) y la estructura [RedirectRequest](#redirectrequest).


>Ejemplo de una estructura payment para un pago:

```shell
{
...
    "payment": {
        "reference": "3210",
        "description": "Pago de prueba 04032019",
        "amount": {
            "currency": "COP",
            "total": "10000"
        }
      },
...
}
```

>Ejemplo de una esrtructura amount para un pago:


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

<aside class="notice">
En el returnUrl se recomienda enviar en la URL un dato con el cual el sitio pueda identificar el registro por ejemplo: <code>https://mysite.com/response/{reference}</code>.
</aside>

* Al recibir la respuesta del <a href="#redirectresponse">redirectResponse</a>, esta debe ser almacenada en su sistema. La operación debe quedar marcada como pendiente. <br>
* Luego de almacenar la respuesta de la operación, el sitio debe redireccionar al usuario sobre la misma pestaña a la interfaz de la sesión en Web Checkout (processUrl) donde debe ingresar la información para procesar el pago:
* Posteriormente el usuario debe seleccionar el medio de pago con el que va realizar la compra.
* Finalmente el usuario ingresa los datos del medio de pago:
* El servicio Web Checkout procesa la operación y retorna a su sistema <code>(returnUrl)</code>. 
* Su sistema [consulta](#consulta) la información de la sesión.
* Su sistema almacena los datos necesarios obtenidos en la respuesta de la sesión.
* Su sistema presenta al usuario la información de la operación.


<aside class="notice">
En caso que la operación quede pendiente, el servicio notificará de forma asíncrona a la URL de notificación una vez sea resuelta (Apruebe o decline)
</aside>


## Pago mixto

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

El pago mixto permite utilizar varios medios de pago al momento de pagar para una misma sesión.

Para hacer uso de esta funcionalidad, en la estructura <code>[payment](#paymentrequest)</code> del <a href="#pagos">Pago</a> es necesario 
enviar el atributo <code>allowPartial</code> como: <code>TRUE</code>.

**Estados adicionales de una sesión de pago mixto**

Al obtener el resultado de la operación, debe tener muy presente los siguiente dos estados:

* <code>APPROVED_PARTIAL:</code> Indica que se realiza al menos un pago aprobado pero no se realizado el pago total de la sesión.
* <code>PARTIAL_EXPIRED:</code> Indica que se realiza al menos un pago aprobado pero no se completó el pago total de la sesión y ésta expiró.

<aside class="notice">
 Es importante identificar según la regla de negocio del sitio al obtener alguno de estos dos estados.
</aside>

## Pago Recurrente

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

Para hacer uso de esta funcionalidad, en la estructura <code>[payment](#paymentrequest)</code> del Pago recurrente es necesario enviar en el objeto <code> [recurring](#recurring)</code> los datos obligatorios para esta estructura, los cuales puede visualizar en la sesión de [Campos usados por tipo de operación](#campos-usados-por-tipo-de-operacion).



<aside class="notice">
Los cobros recurrentes sólo se realizarán en el caso que  la transacción inicial sea aprobada.
</aside>

<aside class="notice">
En el caso de fallar un cobro recurrente, éste será reintentado cada día durante N días, si luego del número máximo de reintentos no se obtiene una transacción aprobada, la reccurencia se cancela.
</aside>

<aside class="notice">
Las recurrencias sólo pueden ser canceladas en la consola administrativa de PlacetoPay.
</aside>

## Suscripción

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

Para hacer uso de esta funcionalidad es necesario enviar en el [RedirectRequest](#redirectrequest) la estructura <code>subscription</code>.

Estructura que se utilizan para el método de pago con suscripción.

* [SubscriptionResponse](#subscriptionresponse)
* [CollectRequest](#collectrequest)
* [NameValuePair](#namevaluepair)
* [Token](#token)
* [Person](#person)
* [Instrument](#instrument)

<aside class="notice">
Se recomienda realizar un cobro por un valor mínimo para identificar que la tarjeta se encuentra activa.
</aside>

<aside class="notice">
La información del token debe ser almacenada en su sitio y relacionada al usuario y/o producto.
Luego de tener el token, puede generar cobros al medio de pago tokenizado usando el servicio <a href="#collect">collect</a>.
</aside>

La estructura de la respuesta contiene toda la información de la petición original y una estructura de suscripción <code>(subscription)</code> con un instrumento <code>(instrument)</code> representado en forma de **token**. [Ver Ejemplo de respuesta de una suscripción](#consulta).

### Recaudar usando suscripción 

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
Para realizar el cobro de una suscripción es necesario usar el método [Collet](#collect) enviando el token obtenido en ésta.

Importante: Este proceso no necesita la intervención del usuario.

<aside class="notice">
Los cobros  se deben realizar en los tiempos acordados con el usuario al momento de la suscripción
</aside>

### Pago recurrente vs Suscripción

  Las principales diferencias entre estas dos funcionalidades son:

  * El pago es ejecutado por PlacetoPay mientras que la suscripción es ejecutada por su sitio. 
  * En un pago recurrente ya creado no se puede modificar los intervalos de tiempo para el cobro ni el monto, mientras que en suscripción la información puede variar dependiendo de cómo envíe la solicitud en el collect.
  * Para eliminar la recurrencia de un pago debe hacerse por la consola administrativa de PlacetoPay, mientras que en suscripción debe invalidar o eliminar de sus sistema el token obtenido.

<aside class="notice">
Si el usuario retorna al comercio, este debe consultar la información de la transacción usando el identificador único <code>requestId</code> generado.
</aside>

# Notificación

>Ejemplo de notificación para el estado de una sesión:

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

Una notificación permite informar al comercio sobre el estado de las sesiones.

Configura en tu servidor una URL de notificación usando los puertos <code>80</code> ó <code>443</code> y en caso de que la sesión se cancele o apruebe, reciba una notificación mediante POST.

Parametro | Tipo | Descripción
--------- | ---- | -----------
requestId <code>Requerido</code> | Int | Identificador único de la sesión.
reference <code>Requerido</code> | String | Reeferencia que acompañada con el <code>requestId</code> permite identificar la sesión de pago afectada.
signature <code>Requerido</code> | String | Firma que permite validar la petición recibida.

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
            "description": "Pago de prueba 04/03/2019",
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

Para consultar la sesión debe enviar el identificador único de la sesión de pago <code>requesId</code> usando el método [GetRequestInformation](#getRequestinformation).

<aside class="notice">
Para este consumo también es necesario enviar en la petición la autenticación.
</aside>

La respuesta de la transacción debe contener diferentes objetos: 

* <a href="#status">status</a>

* [request](#redirectrequest)

* [fields](#redirectrequest)<br>

Datos de la transacción:

Parametro | Tipo | Descripción
--------- | ---- | -----------
internalReference <code>Opcional</code> | Int | Referencia interna en PlacetoPay.
paymentMethod <code>Opcional</code> | String | Código del método de pago utilizado.
paymentMethodName <code>Opcional</code> | String | Nombre del método de pago utilizado.
issuerName <code>Opcional</code> | String | Nombre del emisor o del procesador.