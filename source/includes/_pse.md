## Webservice PSE

A continuación se describen cada uno de los elementos y posibles situaciones que se deben tener presentes para que una transacción por tu sitio con el medio de pago PSE sea exitosa.

### Flujo de integración PSE
> Ejemplo para integrar un pago con PSE:

```php
<?php
     class Pagos
       {
       public function bancos()
       {
              $seed = date('c');
              $secretKey = '024h1IlD';
              $trankey = sha1($seed.$secretKey);

              $servicio = 'https://test.placetopay.com/soap/pse/v11/?wsdl'; //url del servicio
              //AUTENTICACION
              $auth = array(
              'login' => '6dd490faf9cb87a9862245da41170ff2',
              'tranKey' => $trankey,
              'seed' => $seed,
              );

              $arguments = array(
                     'auth' => $auth,
                     );
              $client = new nusoap_client($servicio, array('trace' => true)); // en lugar de SoapClient  se utiliza nusoap_client
              // LAMADA AL METODO GETBANKLIST DE PSE
              $resp = $client->call('getBankList', $arguments);

              $resp1 = $resp['getBankListResult']['item'];

              echo '<h5>Seleccione su banco </h5><br>
               <select name="banco" id="bank" class="form-control" onchange ="r();">';
              // SE RECORRE EL ARRAY PARA LISTAR BANCOS
              foreach ($resp1 as $key => $valor) {
              echo '<option value="'.$valor['bankCode'].'">'.$valor['bankName'].'</option>';
              }
              echo '</select>';
       }

       public function RealizarPago()
       {
              $seed = date('c');
              $secretKey = '024h1IlD';
              $trankey = sha1($seed.$secretKey);
              //PAGO POR PSE
              if (isset($_POST['banco']) && $_POST['banco'] != '') {
              $banco = $_POST['banco'];
              $servicio = 'https://test.placetopay.com/soap/pse/v11/?wsdl'; //url del servicio

              $auth = array(
              'login' => '6dd490faf9cb87a9862245da41170ff2',
              'tranKey' => $trankey,
              'seed' => $seed,
              );
              $payer = array(
                     'documentType' => $_POST['tipdoc'],
                     'document' => $_POST['cedula'],
                     'firstName' => $_POST['nombre'],
                     'lastName' => $_POST['apellido'],
                     'emailAddress' => $_POST['correo'],
                     'city' => $_POST['ciudad'],
                     'province' => $_POST['pais'],
                     'country' => 'Antioquia',
                     'mobile' => $_POST['telefono'],
                     );

              $transaction = array(
                     'bankCode' => $banco,
                     'bankInterface' => 0,
                     'returnURL' => 'http://localhost/API_AIM/Controlador/RespuestaPSE.php',
                     'reference' => $reference = time(),
                     'description' => 'Pago',
                     'language' => 'ES',
                     'currency' => $_POST['moneda'],
                     'totalAmount' => $_POST['total'],
                     'taxAmount' => 0,
                     'devolutionBase' => 0,
                     'tipAmount' => 0,
                     'payer' => $payer,
                     'ipAddress' => '192.168.1.12',
                     );
              $arguments = array(
                     'auth' => $auth,
                     'transaction' => $transaction,
                     );
              $client = new nusoap_client($servicio, array('trace' => true));
              // SE CREA LA TRANSACCION PARA PAGO POR PSE
              $resp = $client->call('createTransaction', $arguments);

              var_dump($resp);

              $_SESSION['transactionId'] = $resp['createTransactionResult']['transactionID'];
              //REDIRECCION AL PSE
              echo '<script>
              window.location = "'.$resp['createTransactionResult']['bankURL'].'";
       </script>';
       }
    }
 }
 
?>
```

*¿Cómo funciona la implementación?*

1. Se muestra como opción de pago PSE (Débitos a cuentas de ahorro y corriente en Colombia).
2. Una vez el usuario la selecciona, se presenta la lista de bancos (debe estar cacheada localmente y su refrescamiento debe ser 1 vez por día) y el tipo de banca a desplegar (personas o empresas).
3. Al tiempo que el cliente confirma la operación, debe consumirse el servicio para realizar la petición de la transacción.
4. Si la petición es exitosa se retorna la URL a la cual debe enviarse al cliente para que realice la operación en el banco. En caso contrario se retorna el motivo del rechazo de la transacción.
5. Almacenar los datos de la transacción retornados por el servicio (identificador de la transacción, autorización/cus, identificador sesión de placetopay).
6. Redirigir el navegador a la URL retornada en caso de éxito.
7. Una vez la transacción retorna a la URL especificada en el consumo de crear transacción, se debe preguntar por el estado de la transacción mediante el consumo un servicio.
8. Dependiendo de la respuesta del servicio, la transacción puede haber sido aprobada, rechazada o seguir pendiente de procesamiento.
9. Informar al usuario el estado de la transacción.
10. Si la transacción está pendiente, o si el cliente al abandonar el portal no ha regresado, se debe tener una sonda que pregunte por el estado de la transacción.

#### Despliegue de bancos disponibles

Una vez el cliente ha determinado que desea pagar con débito a una cuenta corriente o de ahorros, deberá presentarle la lista de los bancos que en el momento están disponibles para la transacción.  Esta lista de bancos se obtiene mediante el consumo del método [getBankList](#getbanklist) el cual debe ser consumido una vez por día, lo que exige que se disponga de un mecanismo de caché que permita recuperar la lista de este si ya ha sido consumido el servicio en el día.

Adicional a darle a escoger con qué banco realizar la transacción, el usuario también debe indicar qué tipo de interfaz del banco desea desplegar, es decir, si la de personas o la de empresas.

Si por algún motivo la lista de bancos no pudo ser obtenida, deberá mostrarse un mensaje “No se pudo obtener la lista de Entidades Financieras, por favor intente más tarde” y, consumir nuevamente el servicio de lista de bancos para intentar obtener dicha lista y poderla almacenar en el caché.

Tenga en cuenta que en operaciones de Multicrédito se debe utilizar el código de servicio para multicrédito y que siempre deberá pasar todas las cuentas dependientes (entidad, servicio) para los créditos así algunas de ellas vayan con valores en cero.  Es requerido que la lista de créditos corresponda con todos los subcódigos dependientes.

#### Confirmación y redirección al banco

Una vez se poseen los datos del pagador y/o comprador, así como la información de qué banco y qué tipo de interfaz debe mostrar el banco deberá entonces consumir el servicio de [createTransaction](#createTransaction) para obtener la URL a la cual deberá redirigir al cliente para finalizar la transacción.

Debe tener en cuenta que en los datos solicitados para crear la transacción se requiere la dirección IP desde la cual el cliente está realizando la transacción, así como la información del agente de navegación usado.  Si lo desea, también puede proveer datos adicionales con la transacción, los cuales le permitirán tener esta información disponible en la consola de consulta de transacciones de Placetopay.

Si la respuesta del servicio es SUCCESS, entonces deberá almacenar la información retornada en su base de datos, de vital importancia el <code>transactionId</code> pues es con este identificador que podrá consultar el estado final de la transacción.

Tenga en cuenta que una respuesta SUCCESS en este punto sólo implica que la transacción ha sido aprovisionada por el banco y está en espera que el cliente llegue a la interfaz del banco, se autentique y autorice la operación iniciada.

Al crear la transacción, esta puede fallar; algunos motivos incluyen montos por fuera del rango permitido, no disponibilidad del banco, problemas de configuración de la cuenta recaudadora, entre otros.  Utilice la propiedad <code>responseReasonText</code> para obtener el mensaje de por qué falló la creación de la transacción.
Algunos códigos no relacionados con el pagador pueden indicarle que genere una alerta sobre problemas con la disponibilidad del servicio, particularmente los relacionados con mala configuración.

Una vez ha almacenado la información en su base de datos y ha confirmado que es exitosa la petición, se redirigirá al cliente a la URL del banco.  Tenga en cuenta que la redirección a la interfaz del banco no debe realizarse en un frame o cualquier otro mecanismo que oculte la URL en la cual el cliente ingresará sus datos de autenticación.

#### Reingreso del cliente

Una vez el cliente ha finalizado la transacción en la interfaz del banco, el banco redirige al cliente a la URL especificada al crear la transacción, en este punto de entrada a su aplicativo, usted deberá consumir el servicio [getTransactionInformation](#gettransactioninformation) que le permite determinar el estado de la transacción. Solo si la transacción tiene como estado OK es porque la transacción fue autorizada, si por el contrario obtiene un estado PENDING es porque aún no tiene respuesta final del banco acerca de la transacción.

Ya sea que el cliente retorne al punto de reingreso y que la transacción esté pendiente, o que haya abandonado la operación y haya roto el flujo normal, se deberá consumir el servicio [getTransactionInformation](#gettransactioninformation) para todas las operaciones que desde que fue invocado el [createTransaction](#createtransaction) llevan más de 7 minutos sin un estado final de operación.

### Métodos de interfaz PSE

A continuación se describen cada una de los métodos de tipo petición respuesta expuestas por el Webservice, se muestran los parámetros de entrada a cada operación y vínculos a las estructuras de datos usadas.

#### getBankList

> Ejemplo para realizar una llamada al método getBankList:

```php
<?php
       public function bancos()
       {
              $seed = date('c');
              $secretKey = '024h1IlD';
              $trankey = sha1($seed.$secretKey);

              $servicio = 'https://test.placetopay.com/soap/pse/v11/?wsdl'; //url del servicio
              //AUTENTICACION
              $auth = array(
              'login' => '6dd490faf9cb87a9862245da41170ff2',
              'tranKey' => $trankey,
              'seed' => $seed,
              );

              $arguments = array(
                     'auth' => $auth,
                     );
              $client = new nusoap_client($servicio, array('trace' => true)); // en lugar de SoapClient  se utiliza nusoap_client
              // LAMADA AL METODO GETBANKLIST DE PSE
              $resp = $client->call('getBankList', $arguments);

              $resp1 = $resp['getBankListResult']['item'];

              echo '<h5>Seleccione su banco </h5><br>
               <select name="banco" id="bank" class="form-control" onchange ="r();">';
              // SE RECORRE EL ARRAY PARA LISTAR BANCOS
              foreach ($resp1 as $key => $valor) {
              echo '<option value="'.$valor['bankCode'].'">'.$valor['bankName'].'</option>';
              }
              echo '</select>';
       }
?>
```

Obtiene la lista de bancos disponibles para el establecimiento de comercio en el sistema PSE de ACH Colombia.

PARÁMETROS

| Nombre | Tipo                              | Descripción            |
|--------|-----------------------------------|------------------------|
| auth   | [Authentication](#authentication-colombia) | Datos de autenticación |

**Retorna** <br>
**[Bank[]](#bank).** Un arreglo con la lista de bancos habilitados.

<aside class="notice">
La respuesta de este servicio debe ser cacheada.
</aside>

<aside class="notice">
El servicio debe ser consumido una vez por día.
</aside>

#### createTransaction

Solicita la creación de una transacción.  En los datos de la solicitud se especifica quién es el pagador, el comprador y el despacho.  Así mismo para cuál de los bancos habilitados se hace la petición y a que URL de retorno debe el banco redirigir al cuentahabiente.

PARÁMETROS

| Nombre      | Tipo                                            | Descripción            |
|-------------|-------------------------------------------------|------------------------|
| auth        | [Authentication](#authentication-colombia)               | Datos de autenticación |
| transaction | [PSETransactionRequest](#psetransactionrequest) | Datos de la solicitud  |

**Retorna** <br>
**[PSETransactionResponse](#psetransactionresponse)**. Respuesta de la creación de la transacción, tenga en cuenta que la URL del banco se entrega solo si la propiedad returnCode es <code>SUCCESS</code>.


#### createTransactionMultiCredit

Solicita la creación de una transacción con dispersión de fondos.  En los datos de la solicitud se especifica quién es el pagador, el comprador y el despacho.  Así mismo para cuál de los bancos habilitados se hace la petición y a que URL de retorno debe el banco redirigir al cuentahabiente.  Así como cada uno de los créditos a aplicar para cada uno de los servicios asociados.  Tenga en cuenta que un servicio multicrédito tiene asociado unos servicios dependientes.  Siempre deberá cobrar a todos los servicios dependientes así el valor para una de los créditos sea cero.


PARÁMETROS

| Nombre      | Tipo                                                                  | Descripción            |
|-------------|-----------------------------------------------------------------------|------------------------|
| auth        | [Authentication](#authentication-colombia)                                     | Datos de autenticación |
| transaction | [PSETransactionMultiCreditRequest](#psetransactionmulticreditrequest) | Datos de la solicitud  |

**Retorna** <br>
**[PSETransactionResponse](psetransactionresponse)**. Respuesta de la creación de la transacción, tenga en cuenta que la URL del banco se entrega solo si la propiedad <code>returnCode</code> es <code>SUCCESS</code>.


#### getTransactionInformation

> Ejemplo para generar una respuesta de una transacción PSE utilizando el método getTransactionInformation:

```php
  <?php
       if (!isset($_SESSION))
       {
       session_start();
       }
              if (isset($_SESSION["transactionId"])) 
              {
              require_once('../nusoap/lib/nusoap.php');
              
              $id = $_SESSION["transactionId"]; //'1476515432' $_SESSION["transactionId"]
                     
              $seed = date('c');
              $secretKey = "024h1IlD";
              $trankey = SHA1($seed.$secretKey);
              $servicio="https://test.placetopay.com/soap/pse/v11/?wsdl"; //url del servicio

              $auth = array(
                     'login' => '6dd490faf9cb87a9862245da41170ff2',
                     'tranKey' => $trankey,
                     'seed' => $seed,
                     );

              $arguments = array(
                            'auth' => $auth,
                            'transactionID'=>$id
                            );
              $client = new nusoap_client($servicio, array('trace' => true));

              $resp = $client->call('getTransactionInformation', $arguments);
              //var_dump($resp);
              foreach ($resp as $key => $valor) 
              {
                     $reason = $valor["responseReasonText"];
                     $reference = $valor["reference"];
                     $estado  =  $valor["transactionState"];
              }
              echo '<div class="card m-t-25 ">
                                   <div class="card-body">
                                          <div class="row">
                                                 <div class="col-md-6">
                                                 <h4>Referencia :'.$reference.' </h4>

                                                 <h4>Estado :'.$estado.' </h4>
                                          
                                                 <h4>Razon :'.$reason.' </h4>
                                                 </div>
                                          </div>
                                   </div>
                            </div>';

              session_destroy();
              }
<?php
```
Obtiene la información de una transacción, debe ser consultado para cualquier operación previamente creada con el método [createTransaction](#createtransaction) o [createTransactionMultiCredit](#createtransactionmulticredit) ya sea cuando regresa del banco o cuando al menos han transcurrido 7 minutos desde que el cliente fue redirigido a la interfaz del banco. Deberá consumirse en intervalos de al menos cada 12 minutos hasta que tenga un estado de transacción <code>diferente</code> a <code>PENDING</code>.

PARÁMETROS

| Nombre        | Tipo                              | Descripción                                                                                                  |
|---------------|-----------------------------------|--------------------------------------------------------------------------------------------------------------|
| auth          | [Authentication](#authentication-colombia) | Datos de autenticación                                                                                       |
| transactionID | int                               | Identificador único de la transacción en Placetopay, equivale al retornado en la creación de la transacción. |


**Retorna** <br>
**[TransactionInformation](#transactioninformation)**. Información del estado de la transacción.


### Estructuras de datos PSE

Las estructuras de datos utilizadas por los métodos del servicio web se describen a continuación:

#### Attribute 

Estructura para almacenar información extendida.

ATRIBUTOS

| Nombre | Tipo        | Descripción                         |
|--------|-------------|-------------------------------------|
| name   | string[30]  | Código para referenciar el atributo |
| value  | string[128] | Valor que asume el atributo         |

#### Persona

> Ejemplo de generación de una estructura Person dentro de la integración de pago con PSE:

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

#### Bank


Estructura para reflejar la información de una entidad bancaria.

ATRIBUTOS

| Nombre   | Tipo       | Descripción                      |
|----------|------------|----------------------------------|
| bankCode | string[4]  | Código de la entidad financiera. |
| bankName | string[60] | Nombre de la entidad financiera. |

#### CreditConcept

Estructura que representa el concepto del crédito a favor de un tercero. 

ATRIBUTOS

| Nombre      | Tipo       | Descripción                                                 |
|-------------|------------|-------------------------------------------------------------|
| entityCode  | string[12] | Código de la entidad del tercero para dispersión.           |
| serviceCode | string[12] | Código del servicio del tercero.                            |
| amountValue | double     | Valor total a recaudar a favor de la entidad.               |
| taxValue    | double     | Discriminación del impuesto aplicado a favor de la entidad. |
| description | string[60] | Descripción el concepto cobrado.                            |

#### PSETransactionRequest

> Ejemplo de generación de una estructura PSETransactionRequest dentro de la integración de pago con PSE:

```php
  <?php
              $transaction = array(
                     'bankCode' => $banco,
                     'bankInterface' => 0,
                     'returnURL' => 'http://localhost/API_AIM/Controlador/RespuestaPSE.php',
                     'reference' => $reference = time(),
                     'description' => 'Pago',
                     'language' => 'ES',
                     'currency' => $_POST['moneda'],
                     'totalAmount' => $_POST['total'],
                     'taxAmount' => 0,
                     'devolutionBase' => 0,
                     'tipAmount' => 0,
                     'payer' => $payer,
                     'ipAddress' => '192.168.1.12',
              );
  ?>
```

Estructura que representa una solicitud de transacción con débitos a cuenta PSE.

ATRIBUTOS

| Nombre         | Tipo                      | Descripción                                                                                                                |
|----------------|---------------------------|----------------------------------------------------------------------------------------------------------------------------|
| bankCode       | string[4]                 | Código de la entidad financiera con la cual realizar la transacción.                                                       |
| bankInterface  | string[1]                 | Tipo de interfaz del banco a desplegar <code>0 = PERSONAS, 1 = EMPRESAS]</code>                                            |
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

#### PSETransactionMultiCreditRequest

Estructura que representa una solicitud de transacción con débitos a cuenta PSE.

ATRIBUTOS

| Nombre         | Tipo                           | Descripción                                                                                                                |
|----------------|--------------------------------|----------------------------------------------------------------------------------------------------------------------------|
| bankCode       | string[4]                      | Código de la entidad financiera con la cual realizar la transacción.                                                       |
| bankInterface  | string[1]                      | Tipo de interfaz del banco a desplegar <code>[0 = PERSONAS, 1 = EMPRESAS]</code>                                           |
| returnURL      | string[255]                    | URL de retorno especificada para la entidad financiera.                                                                    |
| reference      | string[32]                     | Referencia única de pago.                                                                                                  |
| description    | string[255]                    | Descripción del pago.                                                                                                      |
| language       | string[2]                      | Idioma esperado para las transacciones acorde a [ISO 631-1](https://es.wikipedia.org/wiki/ISO_639-1), mayúscula sostenida. |
| currency       | string[3]                      | Moneda a usar para el recaudo acorde a [ISO 4217](https://es.wikipedia.org/wiki/ISO_4217).                                 |
| totalAmount    | double                         | Valor total a recaudar.                                                                                                    |
| taxAmount      | double                         | Discriminación del impuesto aplicado.                                                                                      |
| devolutionBase | double                         | Base de devolución para el impuesto.                                                                                       |
| tipAmount      | double                         | Propina u otros valores exentos de impuesto (tasa aeroportuaria) y que deben agregarse al valor total a pagar.             |
| payer          | [Person](#persona)             | Información del pagador.                                                                                                   |
| buyer          | [Person](#persona)             | Información del comprador.                                                                                                 |
| shipping       | [Person](#persona)             | Información del receptor.                                                                                                  |
| ipAddress      | string[15]                     | Dirección IP desde la cual realiza la transacción el pagador.                                                              |
| userAgent      | string[255]                    | Agente de navegación utilizado por el pagador.                                                                             |
| additionalData | [Attribute[]](#attribute)      | Datos adicionales para ser almacenados con la transacción.                                                                 |
| credits        | [CreditConcept](creditconcept) | Detalle de la dispersión a realizar.                                                                                       |

#### PSETransactionResponse
 
Estructura con la información de respuesta para la creación de una transacción.

ATRIBUTOS

| Nombre             | Tipo        | Descripción                                                                |
|--------------------|-------------|----------------------------------------------------------------------------|
| transactionID      | int         | Identificador único de la transacción en Placetopay.                       |
| sessionID          | string[32]  | Identificador único de la sesión en Placetopay.                            |
| returnCode         | string[30]  | Código de respuesta de la transacción, uno de los siguientes valores:      |
|                    |             | SUCCESS                                                                    |
|                    |             | FAIL_ENTITYNOTEXISTSORDISABLED                                             |
|                    |             | FAIL_BANKNOTEXISTSORDISABLED                                               |
|                    |             | FAIL_SERVICENOTEXISTS                                                      |
|                    |             | FAIL_INVALIDAMOUNT                                                         |
|                    |             | FAIL_INVALIDSOLICITDATE                                                    |
|                    |             | FAIL_BANKUNREACHEABLE                                                      |
|                    |             | FAIL_NOTCONFIRMEDBYBANK                                                    |
|                    |             | FAIL_CANNOTGETCURRENTCYCLE                                                 |
|                    |             | FAIL_ACCESSDENIED                                                          |
|                    |             | FAIL_TIMEOUT                                                               |
|                    |             | FAIL_DESCRIPTIONNOTFOUND                                                   |
|                    |             | FAIL_EXCEEDEDLIMIT                                                         |
|                    |             | FAIL_TRANSACTIONNOTALLOWED                                                 |
|                    |             | FAIL_RISK                                                                  |
|                    |             | FAIL_NOHOST                                                                |
|                    |             | FAIL_NOTALLOWEDBYTIME                                                      |
|                    |             | FAIL_ERRORINCREDITS                                                        |
| trazabilityCode    | string[40]  | Código único de seguimiento para la operación dado por la red ACH          |
| transactionCycle   | int         | Ciclo de compensación de la red                                            |
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
| bankProcessDate    | string      | Fecha de procesamiento de la transacción acorde a ISO 8601(https://www.iso.org/iso-8601-date-and-time-format.html).        |
| onTest             | boolean     | Indicador de si la transacción esta o no en modo de pruebas.                                                               |
| returnCode         | string[30]  | Código de respuesta de la transacción, uno de los siguientes:                                                              |
|                    |             | SUCCESS                                                                                                                    |
|                    |             | FAIL_INVALIDTRAZABILITYCODE                                                                                                |
|                    |             | FAIL_ACCESSDENIED                                                                                                          |
|                    |             | FAIL_INVALIDSTATE                                                                                                          |
|                    |             | FAIL_INVALIDBANKPROCESSINGDATE                                                                                             |
|                    |             | FAIL_INVALIDAUTHORIZEDAMOUNT                                                                                               |
|                    |             | FAIL_INCONSISTENTDATA                                                                                                      |
|                    |             | FAIL_TIMEOUT                                                                                                               |
|                    |             | FAIL_INVALIDVATVALUE                                                                                                       |
|                    |             | FAIL_INVALIDTICKETID                                                                                                       |
|                    |             | FAIL_INVALIDSOLICITEDATE                                                                                                   |
|                    |             | FAIL_INVALIDAUTHORIZATIONID                                                                                                |
|                    |             | FAIL_TRANSACTIONNOTALLOWED                                                                                                 |
|                    |             | FAIL_ERRORINCREDITS                                                                                                        |
|                    |             | FAIL_EXCEEDEDLIMIT                                                                                                         |
| trazabilityCode    | string[40]  | Código único de seguimiento para la operación dado por la red ACH.                                                         |
| transactionCycle   | int         | Ciclo de compensación de la red.                                                                                           |
| transactionState   | string[20]  | Información del estado de la transacción <code>OK, NOT_AUTHORIZED,</code> <code> PENDING, FAILED</code>.                   |
| responseCode       | int         | Estado de la operación en Placetopay.                                                                                      |
| responseReasonCode | string[3]   | Código interno de respuesta de la operación en Placetopay.                                                                 |
| responseReasonText | string[255] | Mensaje asociado con el código de respuesta de la operación en Placetopay.                                                 |