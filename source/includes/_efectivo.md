## Webservice efectivo

En esta sesión se describen cada una de los métodos y estructuras que se deben tener presentes para que una transacción por un sitio que integre el medio de pago efectivo se genere adecuadamente.

### Flujo de integración efectivo

> Ejemplo para integrar un pago en efectivo:

```php
<?php
 
  class Pagos 
  {

      public function efectivo (){

         $seed = date('c');
		 $secretKey = "024h1IlD";
		 $trankey = SHA1($seed.$secretKey);

         $servicio="https://test.placetopay.com/soap/cashorder/?wsdl"; //url del servicio
    
    // CREACION DE LA AUTENTICACION
          $auth = array(
                'login' => '6dd490faf9cb87a9862245da41170ff2',
                'tranKey' => $trankey,
                'seed' => $seed,
            );
            
           $payer = array(
                        'documentType'=>"CC",
                        'document'=>"1037390240",
                        'firstName'=>"Juan",
                        'lastName'=>"Chavarria",
                        'emailAddress'=>"soporte9@placetopay.com",
                        'city'=>"Bello",
                        'province'=>"Colombia",
                        'country'=>"Antioquia",
                        'mobile'=>"3104317467"
                    );


           $transaction = array(
                        'reference'=>"123",
                        'description'=>"prueba efectivo",
                        'language'=>"ES",
                        'currency'=>"COP",
                        'totalAmount'=>1000,
                        'taxAmount'=>0,
                        'subtotalAmount'=>0,
                        'expiration'=>date('c', strtotime('+1 hour')),
                        'buyer'=>$payer,
                        'ipAddress'=>"192.168.1.12",
                        'notificationURL'=>"http://localhost/API_AIM/Controlador/Respuesta.php"
                    );

            $arguments = array(
                                'auth' => $auth,
                                'order'=>$transaction
                            );
            $client = new nusoap_client($servicio,array('trace' => true));
     // SE CREA LA ORDEN PARA EL PAGO EN EFECTIVO
        $resp = $client->call('addOrder', $arguments);
        var_dump($resp);
    }
  }
?>
```

1. El establecimiento invoca el método [addOrder](#addorder) para agregar una orden de pago en efectivo.
2. Si se desea eliminar una orden de pago se puede invocar al método [deleteOrder](#deleteorder), se debe tener en cuenta que la orden no podrá ser eliminada si ya ha sido pagada.
3. Eventualmente se puede invocar al método [flushOrders](#flushorders) que permite purgar las órdenes de pago que ya están vencidas y que no han sido pagadas.
4. Para obtener el PDF con las instrucciones para pagar la orden de efectivo se debe invocar el método [getOrderPDF](#getorderpdf) que retorna un PDF codificado en Base64.
5. Para consultar el estado de una transacción se puede invocar el método [getOrder](#getorder) el cual brinda todos los datos de la orden incluyendo el estado y si está pagada la información de la transacción.


### Métodos de interfaz efectivo

A continuación se describen cada una de los métodos de tipo entrada y salida expuestos por el Webservice efectivo, igualmente se muestran los parámetros de entrada a cada operación y vínculos a las estructuras de datos usadas.

#### addOrder

> Ejemplo para agregar una orden para el pago en efectivo dentro de la integración del mismo:

```php

<?php
...
    $transaction = array(
                    'reference'=>"123",
                    'description'=>"prueba efectivo",
                    'language'=>"ES",
                    'currency'=>"COP",
                    'totalAmount'=>1000,
                    'taxAmount'=>0,
                    'subtotalAmount'=>0,
                    'expiration'=>date('c', strtotime('+1 hour')),
                    'buyer'=>$payer,
                    'ipAddress'=>"192.168.1.12",
                    'notificationURL'=>"http://localhost/API_AIM/Controlador/Respuesta.php"
    );

    $arguments = array(
                        'auth' => $auth,
                        'order'=>$transaction
                    );
    $client = new nusoap_client($servicio,array('trace' => true));
    // SE CREA LA ORDEN PARA EL PAGO EN EFECTIVO
    $resp = $client->call('addOrder', $arguments);

    var_dump($resp);
...
?>
```

Agrega una orden de pago para efectivo.

PARÁMETROS

| Nombre | Tipo                              | Descripción               |
|--------|-----------------------------------|---------------------------|
| auth   | [Authentication](#authentication-colombia) | datos de autenticación    |
| order  | [CashOrder](#cashorder)           | datos de la orden de pago |

**Retorna** <br>
**int**. La referencia única de la orden de pago en Placetopay.

#### getOrder


Obtiene la información de una orden de pago en efectivo para conocer su estado.

PARÁMETROS

| Nombre | Tipo                              | Descripción                                       |
|--------|-----------------------------------|---------------------------------------------------|
| auth   | [Authentication](#authentication-colombia) | Datos de autenticación.                           |
| id     | int                               | Referencia única de la orden de pago a consultar. |

**Retorna** <br>
**[CashOrderInfo](cashorderinfo)**. Información de la orden, si la orden no existe el estado de la orden será NOT_FOUND, si hay un error al consultar el registro se retornará una excepción SoapFault.

#### deleteOrder

Elimina una orden de pago en Placetopay que no haya sido pagada.

PARÁMETROS

Nombre | Tipo | Descripción
auth | [Authentication](#authentication-colombia) | Datos de autenticación.
id | int | Referencia única de la orden de pago a eliminar.

**Retorno** <br>
**int**. Número de registros eliminados, si es un cero indicará que el registro no fue hallado, si hay un error al eliminar el registro se retornará una excepción <code>SoapFault</code>.

#### flushOrders

Purga todas las órdenes de pago no pagadas que ya han cumplido la fecha de corte en Placetopay.

PARÁMETROS

| Nombre | Tipo                              | Descripción             |
|--------|-----------------------------------|-------------------------|
| auth   | [Authentication](#authentication-colombia) | Datos de autenticación. |

**Retorna** <br>
**int**. Número de órdenes de pago eliminadas.

#### getOrderPDF

Genera el PDF para el pago en efectivo en la red de corresponsales que actualmente se permita. Para la fecha de este documento esta red involucra a corresponsales del grupo AVAL/ATH, Baloto, Efecty, cajas de Almacenes Éxito, Carulla, Surtimax y Súper Inter.  Así como cajeros electrónicos del grupo AVAL. Próximamente la red de oficinas de Bancolombia.

PARÁMETROS

| Nombre | Tipo                              | Descripción                                    |
|--------|-----------------------------------|------------------------------------------------|
| auth   | [Authentication](#authentication-colombia) | Datos de autenticación.                        |
| id     | int                               | Referencia única de la orden de pago generada. |

**Retorno** <br>
**base64Binary**. PDF generado codificado como base64, si hay un error al generar el PDF se retornará una excepción <code>SoapFault</code>.


### Estructuras de información efectivo

En este apartado se describen cada una de las estructuras de datos usadas por los métodos del Webservice efectivo.

#### Attribute 

Estructura para almacenar información extendida.

ATRIBUTOS

| Nombre | Tipo        | Descripción                          |
|--------|-------------|--------------------------------------|
| name   | string[30]  | Código para referenciar el atributo. |
| value  | string[255] | Valor que asume el atributo.         |

#### Persona

> Ejemplo de generación de una estructura Person dentro de la integración de pago en efectivo:

```php
<?php
...
    $payer = array(
                        'documentType'=>"CC",
                        'document'=>"1037390240",
                        'firstName'=>"Juan",
                        'lastName'=>"Chavarria",
                        'emailAddress'=>"soporte9@placetopay.com",
                        'city'=>"Bello",
                        'province'=>"Colombia",
                        'country'=>"Antioquia",
                        'mobile'=>"3104317467"
                    );
...  
?>                  
```                

Estructura para reflejar la información de una persona involucrada en una orden de pago.

ATRIBUTOS

| Nombre       | Tipo        | Descripción                                                                                                                                          |
|--------------|-------------|------------------------------------------------------------------------------------------------------------------------------------------------------|
| document     | string[12]  | Número de identificación de la persona.                                                                                                              |
| documentType | string[3]   | Tipo de documento de identificación de la persona.                                                                                                   |
|              |             | CC = Cédula de ciudadanía colombiana                                                                                                                 |
|              |             | CE = Cédula de extranjería                                                                                                                           |
|              |             | TI = Tarjeta de identidad                                                                                                                            |
|              |             | PPN = Pasaporte                                                                                                                                      |
|              |             | NIT = Número de identificación tributaria                                                                                                            |
|              |             | SSN = Social Security Number                                                                                                                         |
| firstName    | string[60]  | Nombres                                                                                                                                              |
| lastName     | string[60]  | Apellidos                                                                                                                                            |
| company      | string[60]  | Nombre de la compañía en la cual labora o representa.                                                                                                |
| emailAddress | string[80]  | Correo electrónico.                                                                                                                                  |
| address      | string[100] | Dirección postal completa.                                                                                                                           |
| city         | string[50]  | Nombre de la ciudad coincidente con la dirección.                                                                                                    |
| province     | string[50]  | Nombre de la provincia o departamento coincidente con la dirección.                                                                                  |
| country      | string[2]   | Código internacional del país que aplica a la dirección física acorde a [ISO 3166-1](https://es.wikipedia.org/wiki/ISO_3166-1), mayúscula sostenida. |
| phone        | string[30]  | Número de telefonía fija.                                                                                                                            |
| mobile       | string[30]  | Número de telefonía móvil o celular.                                                                                                                 |

#### CashOrder

> Ejemplo de generación de una estructura CashOrder dentro de la integración de pago en efectivo:

```php
<?php
...
    $transaction = array(
                    'reference'=>"123",
                    'description'=>"prueba efectivo",
                    'language'=>"ES",
                    'currency'=>"COP",
                    'totalAmount'=>1000,
                    'taxAmount'=>0,
                    'subtotalAmount'=>0,
                    'expiration'=>date('c', strtotime('+1 hour')),
                    'buyer'=>$payer,
                    'ipAddress'=>"192.168.1.12",
                    'notificationURL'=>"http://localhost/API_AIM/Controlador/Respuesta.php"
    );
...
?>
```

Estructura con la información de una orden de pago en efectivo.

ATRIBUTOS

| Nombre          | Tipo                      | Descripción                                                                                                                   |
|-----------------|---------------------------|-------------------------------------------------------------------------------------------------------------------------------|
| reference       | string[32]                | Referencia única de pago.                                                                                                     |
| description     | string[255]               | Descripción o detalle de la orden de pago.                                                                                    |
| language        | string[2]                 | Idioma esperado para las transacciones acorde a [ISO 631-1](https://es.wikipedia.org/wiki/ISO_639-1), usar ES.                |
| currency        | string[3]                 | Código de la moneda acorde a [ISO 4217](https://es.wikipedia.org/wiki/ISO_4217) en la cual está expresado el cobro, usar COP. |
| totalAmount     | decimal                   | Valor total a recaudar.                                                                                                       |
| taxAmount       | decimal                   | Discriminación del impuesto aplicado.                                                                                         |
| subtotalAmount  | decimal                   | Valor base antes de impuestos.                                                                                                |
| expiration      | datetime                  | Fecha y hora máxima hasta la cual se recibe el pago [AAAA-MM-DDTHH:NN:SS]                                                     |
| buyer           | [Person](#persona)        | Información del comprador.                                                                                                    |
| ipAddress       | string                    | Dirección IP desde la cual realiza la solicitud el pagador.                                                                   |
| userAgent       | string                    | Agente de navegación utilizado por el pagador al hacer la solicitud.                                                          |
| additionalData  | [Attribute[]](#attribute) | Datos adicionales para ser almacenados con la transacción.                                                                    |
| notificationURL | string[255]               | URL de notificación una vez la transacción es realizada.                                                                      |

#### Transaction


Estructura con la información de la transacción asociada a la orden de pago.

ATRIBUTOS

| Nombre            | Tipo        | Descripción                 |
|-------------------|-------------|-----------------------------|
| paymentMethod     | string[5]   | Código del medio de pago.   |
| paymentMethodName | string[128] | Nombre del medio de pago.   |
| amount            | decimal     | Valor recaudado.            |
| authorization     | string[40]  | Número de autorización.     |
| receipt           | string[40]  | Número de recibo en la red. |

#### CashOrderInfo

Estructura con la información de una orden de pago en efectivo y la información de la transacción si existe. Hereda todos los atributos de [CashOrder](#cashorder) y agrega las propiedades que se enuncian a continuación.

ATRIBUTOS

| Nombre      | Tipo                        | Descripción                                                                            |
|-------------|-----------------------------|----------------------------------------------------------------------------------------|
| status      | string                      | Estado actual de la orden: NOT_FOUND, PENDING, EXPIRED, PAYED, PAYED_PARTIAL, REFUNDED |
| status_date | dateTime                    | Fecha y hora desde la cual se encuentra en el estado.                                  |
| transaction | [Transaction](#transaction) | Información de la transacción cuando el estado es pagado.                              |
