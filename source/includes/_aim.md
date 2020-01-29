## Webservice Credito Colombia

Aquí se describe un método de conexión a Placetopay basado en el envío de datos por método POST a la plataforma en una comunicación servidor servidor bajo un esquema REST.

Este mecanismo sólo debe ser usado por aquellos clientes de la plataforma que requieren capturar la información del tarjeta habiente incluyendo los datos sensibles. Este tipo de integración sólo está pensado para soportar la operación de tarjetas no presenciales que no requieran PIN para su operación.

### Descripción del método

El método de integración avanzado o AIM, contempla las operaciones básicas para el procesamiento de tarjetas no presenciales, esto es la autorización, captura, reverso, asiento, anulación. A continuación se describirán cada uno de los usos y los datos requeridos en la trama para cada uno de los casos. La respuesta para cada operación está unificada en un solo tipo de trama resultante.
La implementación de este método se hizo pensando en minimizar el ancho de banda requerido para la comunicación así como la facilidad de implementación en cualquier lenguaje de programación que sea capaz de establecer una comunicación por sockets a través del protocolo HTTPS.

### Flujo de integración AIM

*Cómo funciona la implementación?*

1. Se hace una compilación de los datos a ser enviados a la plataforma, incluyendo los datos sensibles de la tarjeta (número, expiración, CVV2).
2. Se prepara la petición POST a nivel de servidor, tenga en cuenta que el servicio espera los datos en codificación [ISO-8859-1](https://es.wikipedia.org/wiki/ISO/IEC_8859-1). Los datos a ser remitidos a Placetopay dependerán de la operación que desea realizar.
3. Se hace el POST a Placetopay, y se espera por la respuesta. Normalmente los tiempos de respuesta son de 2 a 3 segundos, sin embargo tenga en cuenta que dependiendo de congestión de la red u otros factores, el tiempo puede incrementarse. Verifique que en la implementación su script espera al menos hasta 15 o 20 segundos antes de dar un timeout.
4. Si obtuvo una respuesta de la plataforma entonces proceda a desglosarla (la respuesta se entrega como una cadena de caracteres separados por coma) y a identificar los datos relevantes para su sistema, normalmente serán el estado de la transacción, el motivo asociado al estado, la franquicia, el banco emisor, la autorización, el recibo y la identificación de la transacción en Placetopay.
5. Si por el contrario no obtuvo una respuesta válida u obtuvo un timeout deberá seguir el proceso de consulta de la transacción para determinar el estado de la misma.
6. Presente al usuario el resultado de la transacción y active las otras rutas de flujo en su sistema.

#### ¿En qué casos es mejor usar este tipo de integración?

La integración AIM se recomienda únicamente en los casos que no sea factible que el usuario realice la transacción ingresando los datos sensibles en Placetopay o en donde se requiere un control particular de la operación.  Algunos ejemplos son:
* Cobros recurrentes desde una base de datos propietaria.
* Integración con un sistema de audiorespuesta.
* Múltiples transacciones a diferentes recaudadores con la misma información del tarjetahabiente.
* Interfaz móvil propietaria, no basada en HTML.
* Integración con sistemas cuya entrada de los datos sensibles no la realice directamente el tarjetahabiente.

#### ¿Qué obligaciones tengo al usar esta integración?

Tenga en cuenta que al usar este tipo de integración, usted está suministrando todos los datos del comprador incluyendo los datos sensibles de la tarjeta, este solo hecho requiere que la captura de la información sea por un mecanismo seguro que no ponga en peligro los datos del tarjetahabiente. Si la información la captura por una página Web, el punto donde se capturan estos datos debe estar sobre protocolo seguro HTTPS usando certificados de seguridad expedidos por una entidad certificadora acreditada.

Si la captura la realiza por un Call Center, quien obtiene los datos sensibles debe ser un mecanismo de IVR (Interactive Voice Response) o audiorespuesta; esto implicaría un rompimiento en el proceso entre el proceso que realiza el operador con su CRM y el IVR quien captura los datos sensibles. Finalmente el CRM consolida los dos conjuntos de datos y los remite a Placetopay.

Si la captura proviene de una base de datos de tarjetas previamente inscritas, asegúrese que esta información está encriptada por un mecanismo seguro de encriptación como AES o 3DES, desencripte solo en el momento en que construye la trama a ser enviada a Placetopay.

<aside class="warning">
Placetopay tiene servicios de tokenización que pueden ser usados.
</aside>

Tenga en cuenta que como regla general usted no debe almacenar el número de tarjeta, fecha de vencimiento y CVV2. Estos solo pueden tener persistencia durante la solicitud de la autorización, una vez procesada no deben ser almacenados.

Como cualquier proceso de integración, este requerirá una certificación del personal de soporte de Placetopay para revisar temas de funcionamiento, mejores prácticas y usabilidad.

### Metodos de interfaz credito Colombia

A continuación se describen las tramas que deben ser enviadas a Placetopay para que la plataforma procese una operación, de igual forma la trama de respuesta entregada por la plataforma.

#### Solicitud de autorización

Este tipo de solicitud permite realizar un cobro en línea. Esta operación se logra remitiendo el valor <code>AUTH_ONLY</code> en la variable <code>x_type</code>.

#### Solicitud de captura

Este tipo de operación preprocesa la transacción y la deja en estado pendiente. Posteriormente se deberá realizar una operación de asiento o de anulación. Esta operación se logra remitiendo el valor <code>CAPTURE_ONLY</code> en la variable <code>x_type</code>.

#### Solicitud de asiento o anulación de captura

Operación complementaria a la de captura. Tiene como objetivo terminar el cobro de la operación o en su defecto cancelar la solicitud. Esta operación se logra remitiendo los valores de <code>SETTLE o VOID</code> en la variable <code>x_type</code> dependiendo de si se desea asentar o anular.

#### Solicitud de reversa

Una operación de reversa tiene sentido sobre una transacción previamente aprobada. Tenga en cuenta que actualmente hay una restricción para la reversa, y esta solo puede solicitarse el mismo día de la transacción aprobada con un límite de las 10:00pm hora colombiana.
Tenga en cuenta que una transacción de reversa puede declinar y que dependerá del banco emisor dar o no la autorización, igual que lo hace con una autorización.


### Estructura de datos credito

Desde aquí se describen cada uno de los campos que son usados en las diferentes tramas. Se han categorizado acorde al tipo de información que representan; la columna valor posible determina en algunos casos el único valor permitido para el campo.

#### Datos de la especificación

ATRIBUTOS

| Campo | Tipo | Valor posible | Descripción
|-------|------|---------------|--------------
| x_version | String[3] | 2.0 | Versión de la trama a ser usada para comunicarse con Placetopay.
| x_delim_data | String[5] | TRUE | Indicador de que los datos de respuesta van delimitados (el delimitador es la coma).
| x_relay_response | String[5] | FALSE | Indicador de si no espera la respuesta.
| x_language | String[2] | | Código del idioma acorde a ISO 631-1 en que se dan las notificaciones en la plataforma. Por defecto es ES (español).

#### Datos del comercio y operación

ATRIBUTOS

| Campo | Tipo | Valor posible | Descripción
|-------|------|---------------|---------------
| x_login | String[32] |  | Identificador del sitio del comercio en la Plataforma.
| x_tran_key | String[16] |  | Clave para poder enviar una transacción.
| x_test_request | String[5] | TRUE / FALSE | Indicador de si la transacción va en modo de prueba o no. Por defecto es FALSE (producción).
| x_type | String[12] | AUTH_ONLY / CAPTURE_ONLY / CREDIT / SETTLE / VOID | Tipo de operación a realizar.
| x_method | String[2] | CC | Método usado para el procesamiento.
| x_email_merchant | String[5] | TRUE / FALSE | Indicador de si debe enviarse un correo electrónico al comercio cuando haya una transacción exitosa.
| x_email_customer | String[5] | TRUE / FALSE | Indicador de si debe enviarse un correo electrónico al pagador cuando haya una transacción exitosa.
| x_customer_ip | String[15] |  | Dirección IP desde la cual está realizando el pago el usuario pagador.

#### Datos del cobro

ATRIBUTOS

| Campo | Tipo | Valor posible | Descripción
|-------|------|---------------|--------------
| x_invoice_num | String[32] |  | Número de referencia, orden de pedido o factura relacionada.
| x_description | String[255] |  | Descripción de la compra.
| x_currency_code | String[3] |  | Código de la moneda acorde a [ISO 4217](https://es.wikipedia.org/wiki/ISO_4217) en la cual está expresado el cobro.
| x_amount | Decimal [9.2] |  | Valor total de la compra incluyendo impuestos.
| x_tax | Decimal [9.2] |  | Valor total de los impuestos.
| x_amount_base | Decimal [9.2] |  | Valor base de devolución de impuestos (solo aplica para Colombia en los casos en que el impuesto sea del 16% o 10%). Se define como el valor sobre el cual fue calculado el impuesto. Por lo tanto si no hay impuesto, no hay base de devolución.
| x_token | String[64] |  | Token a ser usado, en el caso de que haya tokenizado los datos de la tarjeta.
| x_card_num | String[16] |  | Número de la tarjeta, no requerido si usa token.
| x_card_type | String[1] | C / A / R | Tipo de la tarjeta, no requerido si usa token.
|  |  |  | C = crédito
|  |  |  | A = débito ahorros
|  |  |  | R = débito corriente
| x_exp_date | String[12] |  | Fecha de vencimiento de la tarjeta, no requerido si usa token, se aceptan los siguientes formatos: MMAA, MMAAAA, AAAAMMDD, AAAA/MM/DD, AAAA-MMDD.
| x_card_code | String[4] |  | Dígitos de verificación de la tarjeta (CVV2).
| x_differed | Integer | 1 - 36 | Número de cuotas a las cuales difiere el pago. En su ausencia se supone a una cuota.

#### Datos para cobros recurrentes

Tenga en cuenta que los valores de recurrencia son todos requeridos si el valor del campo <code>x_recurring_bill</code> es <Code>TRUE</code>. Estos datos no tienen sentido en otro caso.

ATRIBUTOS

|Campo|Tipo|Valor posible|Descripción
|-----|----|-------------|-------------
| x_recurring_bill | String[5] | TRUE / FALSE | Indicador de si la transacción es recurrente. En su ausencia se supone <code>FALSE</code>.
| x_recurring_periodicity | String [1] | Y / M / D | Periodicidad para volver a enviar la transacción.
|  |  |  | Y = anual
|  |  |  | M = mensual
|  |  |  | D = diaria
| x_recurring_interval | Integer |  | El intervalo aplicado a la periodicidad, por ejemplo si el intervalo es 3 y la periodicidad es M, entonces se estará haciendo el cobro trimestralmente. En su ausencia se supone uno (1).
| x_recurring_max_periods | |Integer |  | Parte del control de número de veces que se realiza el cobro, corresponde al máximo de veces que sucederá la recurrencia. Si no se desea establecer un máximo debe indicarse -1. Debe especificarse este parámetro o en su defecto <code>x_recurring_due_date.</code>
| x_recurring_due_date | String[10] |  | Parte del control de número de veces que se realiza el cobro, corresponde a la fecha hasta la cual se sucederá la recurrencia.Debe establecerse como una fecha en formato AAAA-MM-DD. Si no se desea establecer un máximo debe indicarse <code>UNLIMITED</code>. Debe especificarse este parámetro o en su defecto <code>x_recurring_max_periods</code>.

#### Datos para cobros con Split

Tenga en cuenta que para pago con dispersión de fondos el comercio debe estar marcado en Placetopay para soportar este tipo de operaciones. De igual forma el <code>x_airline</code> debe llegar en la trama con un código de convenio válido, de lo contrario no hay dispersión y solo se realiza el cobro del valor especificado en <code>x_amount.</code>

ATRIBUTOS

| Campo | Tipo | Valor posible | Descripción
|-------|------|---------------|--------------
| x_airline | Int |  | Código del convenio, cuando se especifica; los valores dados en <code>x_amount</code> serán cobrados a favor de la entidad del convenio. Así mismo se habilita el cobro del valor dado en el <code>x_service_fee</code>.
| x_airport_tax | Decimal [8,2] |  | Valor de la tasa aeroportuaria, solo aplica cuando el convenio es una aerolínea.
| x_service_fee_code | Int | 99 | Código de compensación para cuando hay dispersión de fondos. Identifica que el <code>valor del x_service_fee</code> será a favor del establecimiento que solicitó la transacción.
| x_service_fee | Decimal [8,2] |  | Valor total de la tasa administrativa incluyendo impuestos, este será el valor recaudado a nombre del establecimiento.
| x_service_fee_tax | Decimal [8,2] |  | Valor total del impuesto de la tasa administrativa.
| x_service_fee_base | Decimal [8,2] |  | Valor base de devolución de impuestos para la TA (solo aplica para Colombia en los casos en que el impuesto sea del 16% o 10%).

#### Datos del pagador

ATRIBUTOS

| Campo | Tipo | Valor posible | Descripción
|-------|------|---------------|--------------
|x_cust_id | String [20] |  | Identificación del pagador, está conformado por el tipo de documento y el documento, separados entre ellos por un espacio. Los tipos de documento válidos son:
|  |  |  | CC: Cédula de ciudadanía colombiana
|  |  |  | CE: Cédula de extranjería
|  |  |  | TI: Tarjeta de identidad
|  |  |  | NIT: Número de identificación tributaria
|  |  |  | PPN: Pasaporte
|  |  |  | SSN: Social security number
|  |  |  | Un ejemplo del dato puede ser:
|  |  |  | CC 71000000.
| x_first_name | String[30] |  | Nombres completos.
| x_last_name | String[30] |  | Apellidos completos.
| x_company | String[50] | |  | Compañía para la cual labora.
| x_address | String[80] |  | Dirección de correspondencia.
| x_city | String[30] |  | Ciudad.
| x_state | String[30] |  | Estado, departamento o provincia.
| x_zip | String[10] |  | Código postal.
| x_country | String[2] |  | Código del país acorde a [ISO 3166-1](http://www.iso.org/iso/english_country_names_and_code_elements).
| x_phone | String[30] |  | Número de telefonía fija.
|  |  |  | Prefereriblemente: +indicativo (zona) teléfono.
| x_fax | String[30] |  | Número de fax. 
|  |  |  | Prefereriblemente: +indicativo (zona) teléfono
| x_mobile | String[30] |  | Número de fax / móvil.
|  |  |  | Prefereriblemente: +indicativo (zona) teléfono
| x_email | String[50] |  | Dirección de correo electrónico.

#### Datos del comprador o beneficiario receptor del bien o servicio

ATRIBUTOS

| Campo | Tipo | Valor posible | Descripción
|-------|------|---------------|--------------
| x_ship_to_id | String [20] |  | Identificación del comprador, está conformado por el tipo de documento y el documento, separados entre ellos por un espacio. Los tipos de documento válidos son:
|  |  |  | CC: Cédula de ciudadanía colombiana
|  |  |  | CE: Cédula de extranjería
|  |  |  | TI: Tarjeta de identidad
|  |  |  | NIT: Número de identificación tributaria
|  |  |  | PPN: Pasaporte
|  |  |  | SSN: Social security number
|  |  |  | TAX: Numero generico impositivo
|  |  |  | RC: Registro civil
|  |  |  | Un ejemplo del dato puede ser: CC 71000000.
| x_ship_to_first_name | String[30] |  | Nombres completos.
| x_ship_to_last_name | String[30] |  | Apellidos completos.
| x_ship_to_company | String[50] |  | Compañía para la cual labora.
| x_ship_to_address | String[80] |  | Dirección de correspondencia.
| x_ship_to_city | String[30] |  | Ciudad.
| x_ship_to_state | String[30] |  | Estado, departamento o provincia.
| x_ship_to_zip | String[10] |  | Código postal.
| x_ship_to_country | String[2] |  | Código del país acorde a [ISO 3166-1](http://www.iso.org/iso/english_country_names_and_code_elements).
| x_ship_to_phone | String[30] |  | Número de telefonía fija.
|  |  |  | Prefereriblemente: +indicativo (zona) teléfono
| x_ship_to_mobile | String[30] |  | Número de celular / móvil.
|  |  |  | Prefereriblemente: +indicativo (zona) teléfono
| x_ship_to_email | String[50] |  | Dirección de correo electrónico.

#### Datos para operaciones sobre transacciones

ATRIBUTOS

| Campo | Tipo | Valor posible | Descripción
|-------|------|---------------|------------
| x_franchise | String[5] |  | Código de la franquicia con la cual se hizo el pago.
| x_approval_code | String[10] |  | Código de aprobación entregado por la entidad financiera en la transacción
original.
| x_transaction_id | String[10] |  | Número único de la transacción proveniente de la transacción original.
| x_parent_session | String[32] |  | Identificador interno de la transacción que se desea asentar o anular, corresponde al <code>x_placetopay_id.</code>

#### Datos adicionales para almacenar con la transacción

ATRIBUTOS

|Campo | Tipo | Valor posible | Descripción
|------|------|---------------|------------
| x_additional_data | Array |  | Utilice esta estructura para enviar datos adicionales a Placetopay que desea almacenar conjuntamente con la transacción. Básicamente este campo actua como un diccionario de datos tipo clave => valor.
|  |  |  | La clave tiene restricciones de forma, no puede iniciar con un número y solo debe contener letras no acentuadas, números, guiones medios o guiones de piso. Tiene una longitud máxima de 30 caracteres. El valor tiene una longitud máxima de 255 caracteres.
|  |  |  | Ej:
|  |  |  | <code>x_additional_data[dato1] = “Hola”</code>
|  |  |  | <code>x_additional_data[otroDato] = “Mundo”</code>

### Información adicional Credito Colombia

Estos son datos que se remiten a la plataforma, y no son tomados en cuenta en la transacción.  Pero se retornan en la respuesta.

#### Campos por tipo de operación

Las convenciones para los campos disponibles por tipo de operación son:  <code>R</code> (requerido), <code>O</code> (opcional), <code>C</code> (condicionado a otro campo), <code>-</code> (no aplica). Cuando se especifica condicionado a otro campo por favor vea el detalle de la descripción de campos para entender de cual depende.

| CAMPOS/OPERACIÓN | AUTH_ONLY | CAPTURE_ONLY | CREDIT |SETTLE | VOID
|------------------|-----------|--------------|--------|-------|-----
| x_version | R | R | R | R | R 
| x_delim_data | R | R | R | R | R 
| x_relay_response | R | R | R | R | R
| x_language | O | O | - | - | -
| x_login | R | R | R | R | R
| x_tran_key | R | R | R | R | R
| x_test_request | O | O | - | - | -
| x_type | R | R | R | R | R
| x_method | O | O | - | - | -
| x_email_merchant | O | O | - | - | - 
| x_email_customer | R | O | - | - | -
| x_customer_ip | R | R | - | R | R
| x_invoice_num | R | R | - | - | -
| x_description | O | O | - | - | -
| x_currency_code | O  | O | - | - | -
| x_amount | R | R | - | - | -
| x_tax | R | R | - | - | -
| x_amount_base | R | R | - | - | -
| x_token | C | C |  |  |  |
| x_card_num | C | C | - | - | -
| x_card_type | C | C | - | - |-
| x_exp_date | C | C | - | - | -
| x_card_code | C | C | - | - | - | -
| x_differed | R | R | - | - | -
| x_recurring_bill | O | O | - | - | -
| x_recurring_periodicity | C | C | - | - | -
| x_recurring_interval | C | C | - | - | -
| x_recurring_max_periods | C | C | - | - | -
| x_recurring_due_date | C | C | - | - | -
| x_airline | O | O | - | - | -
| x_airport_tax | C | C | - | - | -
| x_service_fee_code | O | O | - | - | -
| x_service_fee | C | C | - | - | -
| x_service_fee_tax | C | C | - | - | -
| x_service_fee_base | C | C | - | - | -
| x_cust_id | R | R | - | - | -
| x_first_name | R | R | - | - | -
| x_last_name | O | O | - | - | -
| x_company | O | O | - | - | -
| x_address | O | O | - | - | -
| x_city | O | O | - | - | -
| x_state | O | O | - | - | -
| x_zip | O | O | - | - | -
| x_country | O | O | - | - | -
| x_phone | O | O | - | - | -
| En la medida de lo posible se recomienda enviarlo por control de fraude | | | | | |
| x_fax | O | O | - | - | - 
| x_mobile | O | O | - | - | -
| x_email | O | O | - | - | - | 
| x_ship_to_id | O | O | - | - | -
| x_ship_to_first_name | O | O | - | - | -
| x_ship_to_last_name | O | O | - | - | -
| x_ship_to_company | O | O | - | - | -
| x_ship_to_address | O | O | - | - | -
| x_ship_to_city | O | O | - | - | -
| x_ship_to_state | O | O | - | - | -
| x_ship_to_zip | O | O | - | - | -
| x_ship_to_country | O | O | - | - | -
| x_ship_to_phone | O | O | - | - | - 
| x_ship_to_mobile | O | O | - | - |-
| x_ship_to_email | O | O | - | - | -
| x_additional_data | O | O | - | - | -
| x_franchise | - | - | R | R | R
| x_approval_code | - | - | R | - | -
| x_transaction_id | - | - | R | - | -
| x_parent_session | - | - | - | R | R

#### Trama de respuesta

A continuación se detalla la trama de respuesta para cualquiera de las operaciones definidas por el servicio AIM.

La trama de respuesta consiste en una cadena de caracteres separados por coma en el siguiente orden posicional.

| Número | Campo | Tipo | Observaciones
|--------|-------|------|--------------
| 1 | x_response_code | Integer | Respuesta a la transacción solicitada.
|  |  |  | 0: Error
|  |  |  | 1: Aprobada
|  |  |  | 2: Declinada
|  |  |  | 3: Pendiente
| 2 | x_response_subcode | String[5] | Código interno que acompaña la respuesta.
| 3 | x_response_reason_code | String[3] | Código de la razón que por la cual se dió la respuesta.
| 4 | x_response_reason_text | String[255] | Mensaje de la razón por la cual se dio la respuesta.
| 5 | x_approval_code | String[10] | Código de aprobación entregado por la entidad financiera.
| 6 | x_avs_result_code | String[1] | Resultado de la validación de los datos de dirección del comprador.
|  |  |  | Actualmente se retorna
|  |  |  | S: Servicio no soportado
| 7 | x_transaction_id | String[10] | Identificador de la transacción establecido por la red.
| 8 | x_invoice_num |  | Copia del dato recibido
| 9 | x_description |  | Copia del dato recibido
| 10 | x_amount |  | Copia del dato recibido
| 11 | x_method |  | Copia del dato recibido
| 12 | x_type |  | Copia del dato recibido
| 13 | x_cust_id |  | Copia del dato recibido
| 14 | x_first_name |  | Copia del dato recibido
| 15 | x_last_name |  | Copia del dato recibido
| 16 | x_company |  | Copia del dato recibido
| 17 | x_address |  | Copia del dato recibido
| 18 | x_city |  | Copia del dato recibido
| 19 | x_state |  | Copia del dato recibido
| 20 | x_zip |  | Copia del dato recibido
| 21 | x_country |  | Copia del dato recibido
| 22 | x_phone |  | Copia del dato recibido
| 23 | x_fax |  | Copia del dato recibido
| 24 | x_email |  | Copia del dato recibido
| 25 | x_ship_to_first_name |  | Copia del dato recibido
| 26 | x_ship_to_last_name |  | Copia del dato recibido
| 27 | x_ship_to_company |  | Copia del dato recibido
| 28 | x_ship_to_address |  | Copia del dato recibido
| 29 | x_ship_to_city |  | Copia del dato recibido
| 30 | x_ship_to_state |  | Copia del dato recibido
| 31 | x_ship_to_zip |  | Copia del dato recibido
| 32 | x_ship_to_country |  | Copia del dato recibido
| 33 | x_tax |  | Copia del dato recibido
| 34 | x_duty |  | Mantenido por compatibilidad
| 35 | x_freight |  | Mantenido por compatibilidad
| 36 | x_tax_exempt |  | Mantenido por compatibilidad
| 37 | x_po_num |  | Mantenido por compatibilidad
| 38 | x_md5_hash | String[32] | MD5 del valor de <code>md5HashKey</code> + <code>x_login</code> + <code>x_transaction_id</code> + <code>x_amount</code>; el amount va formateado con dos decimales. El hash es un valor establecido en la plataforma. Sirve para comprobar la estabilidad de la trama.
| 39 | x_CVV2_response | String[1] | Indicador de comprobación del CVV2.
|  |  |  | P: No procesado
| 40 | x_CAVV_response | String[1] | Indicador de comprobación del CAVV.
| 41 | x_bank_currency | String[3] | Código de la moneda con la cual se hizo el recaudo.
| 42 | x_bank_factor | Decimal | Tasa de conversión de la moneda original vs la moneda de recaudo.
| 43 | x_bank_amount | Decimal | Valor recaudado convertido a la moneda esperada por la entidad financiera.
| 44 | x_franchise | String[5] | Código de la franquicia con la cual se hizo el pago.
|  |  |  | CR_CR => Credencial
|  |  |  | CR_DN => Diners
|  |  |  | CR_AM => AMEX
|  |  |  | CR_VS => Visa
|  |  |  | CR_MC => Mastercard
| 45 | x_bank_name | String[30] | Nombre del banco que autorizó la transacción (no siempre disponible)
| 46 | x_placetopay_id | String[32] | Identificador único de la transacción retornado por Placetpay
| 47 | x_ta_response_code |  |  
| 48 | x_ta_response_reason_code |  |
| 49 | x_ta_approval_code |  |  
| 50 | x_ta_transaction_id |  |
| 51 | x_placetopay_internal_reference | Int | Referencia interna en Placetopay enviada a la red como referencia de la transacción
| 52 | x_response_reason_code_b24 | String[3] | Código BASE24 equivalente al código ISO que llega en <code>x_response_reason_code</code>
| 53 | x_truncated_pan | String[16] | Número de tarjeta de crédito truncado admisible para ser visualizado
| 54 | x_discount_code | String[32] | Código de descuento aplicado.  Un valor en blanco en este campo hace que los datos descuento de los demás campos tampoco deban tenerse en cuenta.
| 55 | x_discount_type | String[20] | Tipo de descuento aplicado.
|  |  |  | MERCHANT => Provisto por el comercio
|  |  |  | FRANCHISE => Provisto por el medio de pago
|  |  |  | OTHER => Provisto por otro actor
| 56 | x_discount_amount | Decimal | Valor total del descuento en la misma moneda de la solicitud
| 57 | x_discount_base | Decimal | Valor base sobre el cual fue aplicado el descuento
| 58 | x_discount_percent | Decimal | Valor porcentual equivalente del descuento
| 59 |  |  | De la posición 59 a 68 se reserva para usos futuros.
| 69 |  |  | A partir de esta posición se retransmiten los datos adicionales recibidos del comercio.

#### Envío de transacciones de prueba

En la plataforma, los comercios pueden estar en pruebas o producción. Dependiendo de este modo preconfigurado tiene sentido el parámetro <code>x_test_request</code>; así mismo como las respuestas obtenidas.

|Configuración del comercio | Configuración del x_test_request | Modo de transacción
|---------------------------|----------------------------------|--------------------
| pruebas | TRUE | pruebas
| pruebas | FALSE | pruebas
| producción | TRUE | pruebas
| producción | FALSE | producción

En modo de pruebas únicamente son aceptados los siguientes números de tarjetas:

| Franquicia | Tarjeta | Comportamiento 
|------------|---------|---------------
| Visa | 4005580000000040 | Rechaza
| Visa | 4007000000027 | Autoriza
| Visa | 4111111111111111 | Autoriza
| Visa | 4212121212121214 | Deja la operación pendiente como si fuera modo captura, la operación se debe asentar o anular por el panel de Placetopay o en su defecto mediante una operación **SETTLE** o **VOID**.
| Visa | 4666666666666669 | Se demora en autorizar 3 minutos. La idea es simular un timeout en su autorización.  Así que el consumo del servicio fallará por tiempo, lo cual obligará al consumo de la sonda para verificar cuando la operación culmina su procesamiento.  Tenga en cuenta los tiempos de consumo del webservice.
| MasterCard | 5424000000000015 | Autoriza
| MasterCard | 5406251000000008 | Autoriza
| AmericanExpress | 370000000000002 | Autoriza
| Diners | 36018623456787 | Autoriza
| BBVA Club Campestre | 8130010000000000 | Autoriza

### Consumo del Webservice de sonda

Para aquellas transacciones que presentan una falla en los tiempos de respuesta, o que no puede ser verificado el <code>x_md5_hash</code>, se deberá consumir un Webservice para obtener de primera mano la información de la transacción.

En los casos que la transacción tiene respuesta pendiente, la regla general es que debe consumirse el servicio cada 12 minutos para las operaciones que tengan al menos 7 minutos de antigüedad. Esto implica que debe existir un proceso automático solicitando al webservice las operaciones que llevan más de 7 minutos pendientes y que el barrido debe hacerse cada 12 minutos.

<aside class="notice">
Tenga en cuenta que en el proceso de certificación, se hará revisión de los consumos de sonda para que concuerden con los tiempos definidos.
</aside>

<aside class="notice">
El metodo de autenticación para el webservice de sonda se mantiene.
</aside>

#### QueryTransaction

Consulta el estado de una transacción acorde a los parámetros usados, tenga en cuenta que la respuesta es un arreglo de transacciones.

URL del servicio:
Pruebas:	[https://test.placetopay.com/soap/placetopay/v11/](https://test.placetopay.com/soap/placetopay/v11/)
Producción:	[https://api.placetopay.com/soap/placetopay/v11/](https://api.placetopay.com/soap/placetopay/v11/)

PARÁMETROS

| Nombre | Tipo | Descripción
|--------|------|------------
| auth | [Authentication](#authentication) | Datos de autenticación.
| request | [QueryRequest](#queryrequest) | Parámetros para buscar la transacción.

**Retorna** <br>
**Transaction[]**. Transacciones que concuerdan con la solicitud.

#### QueryRequest

Estructura para solicitar la búsqueda de una transacción.

PARÁMETROS

| Nombre | Tipo | Descripción
|--------|------|------------
| reference | String[32] | Número de referencia, orden de compra o factura relacionada. (<code>x_invoice_num</code> usado).
| currency | String[3] | Moneda utilizada en la solicitud acorde a [ISO 4217](https://www.currency-iso.org/dam/downloads/lists/list_one.xml).
| totalAmount | double | Valor total de la operación, si es de aerolinea con TA debe ser la suma del valor del tiquete, la tasa aeroportuaria y la tarifa administrativa.
|  |  |  | <code>x_amount + x_airport_tax + x_service_fee</code>.
| requestDate | String | Fecha de solicitud o creación de la transacción acorde a [ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html), se obvia la hora de la solicitud

#### Transaction (Credito)

Estructura con la información de una transacción.

PARÁMETROS

| Nombre | Tipo | Descripción
| -------|------|------------
| transactionID | int | El ID de referencia pasado al procesador.
| sessionID | string | Identificador único de la sesión en Placetopay.
| reference | string | La referencia de la transacción.
| requestDate | string | Fecha de solicitud o creación de la transacción acorde a [ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html).
| bankProcessDate | string | Fecha de procesamiento de la transacción acorde a [ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html).
| onTest | boolean | Indicador de si la transacción es en modo de pruebas o no.
| description | string | El dato pasado como x_description.
| currency | string | El dato pasado como <code>x_currency</code>.
| totalAmount | double | Valor total de la compra.
| taxAmount | double | Impuesto asociado a la compra.
| devolutionBase | double | Base de devolución del IVA.
| tipAmount | double | El valor de la tasa aeroportuaria: <code>x_airport_tax</code>.
| airline | int | Código de la aerolínea.
| serviceFee | double | Valor total de la tasa administrativa.
| serviceFeeTax | double | Impuesto asociado a la tasa administrativa.
| serviceFeeBase | double | Base de devolución del IVA asociado a la tasa administrativa.
| payer | [Person](#person) | Información del pagador.
| buyer | [Person](#person) | Información del comprador (<code>x_shipping_fields</code>)
| shipping | [Person](#person) | Información del receptor, no usado actualmente.
| ipAddress | string | Dirección IP desde la cual realiza la transacción el pagador.
| userAgent | string | Agente de navegación (browser) utilizado por el pagador.
| franchise | string | El código de la franquicia usada.
| franchiseName | string | Nombre de la franquicia.
| bankName | string | Nombre del banco del tarjeta o cuenta habiente.
| bankCurrency | string | Moneda aceptada por el banco.
| bankFactor | float | Factor de conversión de la moneda.
| authorization | string | Número de autorización.
| receipt | string | Número de recibo.
| refunded | boolean | Indicador de si la transacción fue reversada.
| transactionState | string | Información del estado de la transacción [<code>OK</code>, <code>NOT_AUTHORIZED</code>, <code>PENDING</code>, <code>FAILED</code>]
| responseCode | int | Estado de la operación:
|  |  | 0 = Fallida
|  |  | 1 = Aprobada
|  |  | 2 = Rechazada 
|  |  | 3 = Pendiente
| responseReasonCode | string | Código interno de respuesta de la operación en Placetopay.
| responseReasonText | string | Mensaje asociado con el código de respuesta de la operación en Placetopay.
| serviceFeeTransactionState | string | Información del estado de la transacción de la tasa administrativa [<code>OK</code>, <code>NOT_AUTHORIZED</code>, <code>PENDING</code>, <code>FAILED</code>]
| serviceFeeResponseCode | int | Estado de la operación de tasa administrativa:
|  |  | 0 = Fallida
|  |  | 1 = Aprobada
|  |  | 2 = Rechazada 
|  |  | 3 = Pendiente
| serviceFeeResponseReasonCode | string | Código interno de respuesta de la operación en Placetopay para la tasa administrativa.
| serviceFeeResponseReasonText | string | Mensaje asociado con el código de respuesta de la operación en Placetopay para la tasa administrativa.
| serviceFeeAuthorization | string | Número de autorización de la tasa administrativa.
| serviceFeeReceipt | string | Número de recibo de la tasa administrativa.
| additional | Attribute[] | Datos adicionales proporcionados con la transacción.
