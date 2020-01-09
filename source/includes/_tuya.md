## Webservice Tuya

A continuación se describe el flujo, metodos e interfaces básicas con las cuales debe contar su desarrollo para poder lograr exitosamente la revisión del proceso.

### Flujo de integración Tuya

> Ejemplo para integrar un pago con tarjeta TUYA O EXITO:

```php
<?php
     class Pagos
       {
       public function RealizarPago()
       {
              $seed = date('c');
              $secretKey = '024h1IlD';
              $trankey = sha1($seed.$secretKey);

              $servicio = 'https://test.placetopay.com/soap/tuya/v11/?wsdl'; //url del servicio
              
              $auth = array(
              'login' => '6dd490faf9cb87a9862245da41170ff2',
              'tranKey' => $trankey,
              'seed' => $seed,
              );
               $payer = array(
                    'documentType' => 'CC',
                    'document' => '1037390240',
                    'firstName' => 'Juan',
                    'lastName' => 'Chavarria',
                    'emailAddress' => 'soporte9@placetopay.com',
                    'city' => 'Bello',
                    'province' => 'Colombia',
                    'country' => 'Antioquia',
                    'mobile' => '3104317467',
                );

                $transaction = array(
                    'franchise' => $tarjetaTuya, //TY_AK,TY_EX
                    'returnURL' => 'http://localhost/API_AIM/Controlador/Respuesta.php',
                    'reference' => '123',
                    'description' => 'prueba venta',
                    'language' => 'ES',
                    'currency' => 'COP',
                    'totalAmount' => 1000,
                    'taxAmount' => 0,
                    'devolutionBase' => 0,
                    'tipAmount' => 0,
                    'payer' => $payer,
                    'buyer' => $payer,
                    'ipAddress' => '192.168.1.12',
                  );
                 $arguments = array(
                    'auth' => $auth,
                    'transaction' => $transaction,
                     );
              $client = new nusoap_client($servicio, array('trace' => true));
             // SE CREA TRANSACCION PARA PAGO POR TARJETAS TUYA
             $resp = $client->call('createTransaction', $arguments);
             echo '<div class="container">
              <div class="card m-t-25 ">
                  <div class="card-body">
                    <br>
                        <strong>ESTADO: </strong>'.$resp['createTransactionResult']['returnCode'].'
                          <br>
                          <strong>RAZON: </strong>'.$resp['createTransactionResult']['responseReasonText'].'
                          <br>
                          <strong>BANCO: </strong>'.$resp['createTransactionResult']['bankName'].'
                          <br>
                          <strong>ID TRANSACCION: </strong>'.$resp['createTransactionResult']['transactionID'].'
                          <br>
                          <strong>URL DEL BANCO: </strong>'.$resp['createTransactionResult']['bankURL'].'
                  <div/>
              <div/>
          <div/>';
      }
  }
?>
```

*¿Cómo funciona la implementación?*

1. Se muestra como opción de pago Tarjeta Éxito y Tarjeta Alkosto (dependerá de si se tiene convenio para cada uno de los medios de pago).
2. Una vez el usuario la selecciona el medio de pago y confirma la operación, debe consumirse el servicio para realizar la petición de la transacción.
3. Si la petición es exitosa se retorna la URL a la cual debe enviarse al cliente para que realice la operación en en medio de pago. En caso contrario se retorna el motivo del rechazo de la transacción.
4. Almacenar los datos de la transacción retornados por el servicio (identificador de la transacción, autorización, identificador sesión de placetopay).
5. Redirigir el navegador a la URL retornada en caso de éxito.
6. Una vez la transacción retorna (retorna a la URL especificada en el consumo del crear transacción), se debe preguntar por el estado de la transacción mediante el consumo un servicio.
7. Dependiendo de la respuesta del servicio, la transacción puede haber sido aprobada, rechazada o seguir pendiente de procesamiento.
8. Informar al usuario el estado de la transacción.
9. Si la transacción está pendiente, o si el cliente al abandonar el portal no ha regresado, se debe tener una sonda que pregunte por el estado de la transacción.

#### Confirmación y redirección a banco
Una vez ya se tengan los datos del pagador y comprador, así como la información de qué medio de pago usará, deberá entonces consumir el servicio de [createTransaction](#createtransaction) para obtener la URL a la cual deberá redirigir al cliente para finalizar la transacción.

Debe tener en cuenta que en los datos solicitados para crear la transacción se requiere la dirección IP desde la cual el cliente estará realizando la transacción, así como la información del agente de navegación usado. Si lo desea, también puede proveer datos adicionales con la transacción, los cuales le permitirán tener esta información disponible en la consola de consulta de transacciones de Placetopay. Tenga en cuenta que el documento del pagador es el documento del dueño de la tarjeta y que no podrá ser modificado posteriormente en la interfaz de *TUYA* para ese medio de pago.

Si la respuesta del servicio es <code>SUCCESS</code>, entonces deberá almacenar la información retornada en su base de datos, de vital importancia el <code>transactionID</code> pues es con este identificador que podrá consultar el estado final de la transacción.

Tenga en cuenta que una respuesta <code>SUCCESS</code> en este punto sólo implica que la transacción ha sido aprovisionada por *TUYA* y está en espera que el cliente llegue a la interfaz del medio de pago, se autentique y autorice la operación iniciada.

Al crear la transacción, esta puede fallar; algunos motivos incluyen montos por fuera del rango permitido, no disponibilidad del medio de pago, problemas de configuración de la cuenta recaudadora, entre otros. Utilice el parametro <code>responseReasonText</code> para obtener el mensaje de por qué falló la creación de la transacción. Algunos códigos no relacionados con el pagador pueden indicarle que genere una alerta sobre problemas con la disponibilidad del servicio, particularmente los relacionados con mala configuración.

Una vez ha almacenado la información en su base de datos y ha confirmado que es exitosa la petición, se redirigirá al cliente a la URL del medio de pago. Tenga en cuenta que la redirección a la interfaz del medio de pago no debe realizarse en un frame o cualquier otro mecanismo que oculte la URL en la cual el cliente ingresará sus datos de autenticación.

#### Reingreso cliente

Una vez el cliente ha finalizado la transacción en la interfaz del banco, el banco redirige al cliente a la URL especificada al crear la transacción, en este punto de entrada a su aplicativo, usted deberá consumir el servicio [getTransactionInformation](#gettransactioninformation) que le permite determinar el estado de la transacción. Solo si la transacción tiene como estado <code>OK</code> es porque la transacción fue autorizada, si por el contrario obtiene un estado <code>PENDING</code> es porque aún no tiene respuesta final del banco acerca de la transacción.

Ya sea que el cliente retorne al punto de reingreso y que la transacción esté pendiente, o que haya abandonado la operación y haya roto el flujo normal, se deberá consumir el servicio [getTransactionInformation](#gettransactioninformation) para todas las operaciones que desde que fue invocado el [createTransaction](#createtransaction) llevan más de 7 minutos sin un estado final de operación.

### Métodos de interfaz Tuya

Los métodos son las operaciones expuestas por el webservice de tipo petición respuesta, donde se muestran los parámetros de entrada y vinculos a las estructuras de datos usadas.

#### createTransaction 
> Ejemplo para utilizar el metodo createTransaction:

```php
<?php
      ...  
        $arguments = array(
              'auth' => $auth,
              'transaction' => $transaction,
          );
        $client = new nusoap_client($servicio, array('trace' => true));
       // SE CREA TRANSACCION PARA PAGO POR TARJETAS TUYA
        $resp = $client->call('createTransaction', $arguments);
       ...
?>
```
Solicita la creación de una transacción.  En los datos de la solicitud se especifica quien es el pagador, el comprador y el despacho. Así mismo para cuál de los medios de pago habilitados se hace la petición y a que URL de retorno debe el medio de pago redirigir al tarjeta habiente.

PARÁMETROS

| Nombre      | Tipo                                              | Descripción            |
|-------------|---------------------------------------------------|------------------------|
| auth        | [Authentication](#authentication)                 | Datos de autenticación |
| transaction | [TuyaTransactionRequest](#tuyatransactionrequest) | Datos de la solicitud  |

**Retorna** <br>
**[TuyaTransactionResponse](#tuyatransactionresponse)**. Respuesta de la creación de la transacción, tenga en cuenta que la URL del medio de pago se entrega solo si la propiedad <code>returnCode</code> es <code>SUCCESS</code>.

#### getTransactionInformation

Obtiene la información de una transacción, debe ser consultado para cualquier operación previamente creada con el método createTransaction ya sea cuando regresa del medio de pago o cuando al menos han transcurrido 7 minutos desde que el cliente fue redirigido a la interfaz del medio de pago. Deberá consumirse en intervalos de al menos cada 12 minutos hasta que tenga un estado de transacción <code>transactionState</code> diferente a <code>PENDING</code>.

PARÁMETROS

| Nombre        | Tipo                              | Descripción                                                                                                  |
|---------------|-----------------------------------|--------------------------------------------------------------------------------------------------------------|
| auth          | [Authentication](#authentication) | Datos de autenticación                                                                                       |
| transactionID | int                               | Identificador único de la transacción en Placetopay, equivale al retornado en la creación de la transacción. |


**Retorna** <br>
**[TransactionInformation](#transactioninformation)**. Información del estado de la transacción.

### Estructuras de datos Tuya

Cada una de las estructuras de datos usadas por los métodos del Webservice se describen a continuación.

#### Attribute 

Estructura para almacenar información extendida.

ATRIBUTOS

| Nombre | Tipo        | Descripción                         |
|--------|-------------|-------------------------------------|
| name   | string[30]  | Código para referenciar el atributo |
| value  | string[128] | Valor que asume el atributo         |

#### Persona

> Ejemplo de generación de una estructura Person dentro de la integración de pago con tarjeta TUYA:

```php
  <?php
       $payer = array(
                    'documentType' => 'CC',
                    'document' => '1037390240',
                    'firstName' => 'Juan',
                    'lastName' => 'Chavarria',
                    'emailAddress' => 'soporte9@placetopay.com',
                    'city' => 'Bello',
                    'province' => 'Colombia',
                    'country' => 'Antioquia',
                    'mobile' => '3104317467',
                );
  ?>
```
Estructura para reflejar la información de una persona involucrada en una transacción.

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

#### TuyaTransactionRequest
> Ejemplo de  una estructura TuyaTransactionRequest:

```php
<?php
  ...
      $transaction = array(
        'franchise' => $tarjetaTuya, //TY_AK,TY_EX
        'returnURL' => 'http://localhost/API_AIM/Controlador/Respuesta.php',
        'reference' => '123',
        'description' => 'prueba venta',
        'language' => 'ES',
        'currency' => 'COP',
        'totalAmount' => 1000,
        'taxAmount' => 0,
        'devolutionBase' => 0,
        'tipAmount' => 0,
        'payer' => $payer,
        'buyer' => $payer,
        'ipAddress' => '192.168.1.12',
      );
  ...
?>
```

Estructura que representa una solicitud de transacción con tarjetas expedidas por TUYA.

ATRIBUTOS

| Nombre         | Tipo                      | Descripción                                                                                                                |
|----------------|---------------------------|----------------------------------------------------------------------------------------------------------------------------|
| franchise      | string[4]                 | Código del medio de pago <Code>TY_EX = Tarjeta Éxito, TY_AK = Tarjeta Alkosto</code>                                       |
| returnURL      | string[255]               | URL de retorno especificada para la entidad financiera.                                                                    |
| reference      | string[32]                | Referencia única de pago.                                                                                                  |
| description    | string[255]               | Descripción del pago.                                                                                                      |
| language       | string[2]                 | Idioma esperado para las transacciones acorde a [ISO 631-1](https://es.wikipedia.org/wiki/ISO_639-1), mayúscula sostenida. |
| currency       | string[3]                 | Moneda a usar para el recaudo acorde a [ISO 4217](https://es.wikipedia.org/wiki/ISO_4217).                                 |
| totalAmount    | double                    | Valor total a recaudar.                                                                                                    |
| taxAmount      | double                    | Discriminación del impuesto aplicado.                                                                                      |
| devolutionBase | double                    | Base de devolución para el impuesto.                                                                                       |
| tipAmount      | double                    | Propina u otros valores exentos de impuesto (tasa aeroportuaria) y que deben agregarse al valor total a pagar.             |
| payer          | [Person](#persona)        | Información del pagador.                                                                                                   |
| buyer          | [Person](#persona)        | Información del comprador.                                                                                                 |
| shipping       | [Person](#persona)        | Información del receptor.                                                                                                  |
| ipAddress      | string[15]                | Dirección IP desde la cual realiza la transacción el pagador.                                                              |
| userAgent      | string[255]               | Agente de navegación utilizado por el pagador.                                                                             |
| additionalData | [Attribute[]](#attribute) | Datos adicionales para ser almacenados con la transacción.                                                                 |
#### TuyaTransactionResponse

Estructura con la información de respuesta para la creación de una transacción.

ATRIBUTOS

| Nombre             | Tipo        | Descripción                                                                |
|--------------------|-------------|----------------------------------------------------------------------------|
| transactionID      | int         | Identificador único de la transacción en Placetopay.                       |
| sessionID          | string[32]  | Identificador único de la sesión en Placetopay.                            |
| returnCode         | string[30]  | Código de respuesta de la transacción, uno de los siguientes valores:      |
|                    |             | SUCCESS                                                                    |
|                    |             | FAIL_INVALIDGATEWAYID                                                      |
|                    |             | FAIL_INVALIDFRANCHISE                                                      |
|                    |             | FAIL_INVALIDRETAILCODE                                                     |
|                    |             | FAIL_INVALIDCURRENCY                                                       |
|                    |             | FAIL_INVALIDAMOUNT                                                         |
|                    |             | FAIL_TIMEOUT                                                               |
|                    |             | FAIL_TRANSACTIONNOTALLOWED                                                 |
|                    |             | FAIL_RISK                                                                  |
|                    |             | FAIL_NOHOST                                                                |
| trazabilityCode    | string[64]  | Código único de seguimiento para la operación dado por la red TUYA         |
| bankCurrency       | string[3]   | Moneda aceptada por el banco acorde a [ISO 4217](https://es.wikipedia.org/wiki/ISO_4217). |
| bankFactor         | float       | Factor de conversión de la moneda.                                         |
| bankURL            | string[255] | URL a la cual remitir la solicitud para iniciar la interfaz del banco, sólo disponible cuando <code>returnCode = SUCCESS</code>. |
| responseCode       | int         | Estado de la operación en Placetopay  <code>0 = FAILED, 1 = APPROVED,</code> <code>2 = DECLINED, 3 = PENDING</code>. |
|                    |             |
| responseReasonCode | string[3]   | Código interno de respuesta de la operación en Placetopay.                 |
| responseReasonText | string[255] | Mensaje asociado con el código de respuesta de la operación en Placetopay. |

#### TransactionInformation

Estructura con la respuesta a una solicitud de información de transacción.

ATRIBUTOS

| Nombre             | Tipo        | Descripción                                                                                                                |
|--------------------|-------------|----------------------------------------------------------------------------------------------------------------------------|
| transactionID      | int         | Identificador único de la transacción en Placetopay.                                                                       |
| sessionID          | string[32]  | Identificador único de la sesión en Placetopay.                                                                            |
| reference          | string[32]  | Referencia única de pago.                                                                                                  |
| requestDate        | string      | Fecha de solicitud o creación de la transacción acorde a ISO 8601(https://www.iso.org/iso-8601-date-and-time-format.html). |
| franchise          | string[5]   | Franquicia con la cual se procesó la transacción.                                                                          |
| bankName           | string[80]  | Nombre del banco o entidad financiera.                                                                                     |
| bankProcessDate    | string      | Fecha de procesamiento de la transacción acorde a ISO 8601(https://www.iso.org/iso-8601-date-and-time-format.html).        |
| onTest             | boolean     | Indicador de si la transacción esta o no en modo de pruebas.                                                               |
| returnCode         | string[30]  | Código de respuesta de la transacción, uno de los siguientes:                                                              |
|                    |             | SUCCESS                                                                                                                    |
|                    |             | FAIL_INVALIDGATEWAYID                                                                                                      |
|                    |             | FAIL_INVALIDFRANCHISE                                                                                                      |
|                    |             | FAIL_INVALIDRETAILCODE                                                                                                     |
|                    |             | FAIL_INVALIDCURRENCY                                                                                                       |
|                    |             | FAIL_INVALIDAMOUNT                                                                                                         |
|                    |             | FAIL_TIMEOUT                                                                                                               |
|                    |             | FAIL_TRANSACTIONNOTALLOWED                                                                                                 |
| authCode           | string[40]  | Código de autorización asignado a la transacción.                                                                          |
| trazabilityCode    | string[64]  | Código único de seguimiento para la operación dado por la red TUYA.                                                        |
| lastDigits         | string[6]   | Últimos dígitos de la tarjeta de crédito usada                                                                             |
| transactionState   | string[20]  | Información del estado de la transacción <code>OK, NOT_AUTHORIZED,</code> <code> PENDING, FAILED</code>.                   |
| responseCode       | int         | Estado de la operación en Placetopay.                                                                                      |
| responseReasonCode | string[3]   | Código interno de respuesta de la operación en Placetopay.                                                                 |
| responseReasonText | string[255] | Mensaje asociado con el código de respuesta de la operación en Placetopay.                                                 |