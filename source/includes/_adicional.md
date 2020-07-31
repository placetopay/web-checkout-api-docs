## Información adicional

### Campos usados por tipo de operación
Matriz con los campos disponibles para enviar por operación.
Las convenciones son: <code>R</code> (requerida), <code>O</code> (opcional), <code>A</code> (recomendada), <code>-</code> (no aplica), <code>R*</code> (Es requerido, se debe enviar uno de ellos)

        |Payment | Subscription | Payment Partial | Reverse Payment | Recurrent payment |Invalidate token | Collect
--------|--------|--------------|-----------------|-----------------|-------------------|-----------------|--------
**locale** | O | O | O | - | O | - | O 
**payer** | O | O | O | - | O | - | R
documentType | O | O | O | - | O | - | R
document | O | O | O | - | O | - | R
name | O | O | O | - | O | - | R 
surname | O | O | O | - | O | - | R
company | O | O | O | - | O | - | O
email | O | O | O | - | O | - | R
address | O | O | O | - | O | - | A
mobile | O | O | O | - | O | - | A
**buyer** | A | A | A | - | A | - | O
documentType | O | O | O | - | O | - | O
document | O | O | O | - | O | - | O
name | A | A | A | - | A | - | O
surname | O | O | O | - | O | - | O
company | O | O | O | - | O | - | O
email | A | A | A | - | A | - | O 
address | O | O | O | - | O | - | O
mobile | O | O | O | - | O | - | O
**payment** | R | - | R | - | R | - | R
reference | R | - | R | - | R | - | R
description | R | - | R | - | R | - | R
amount | R | - | R | - | R | - | R
<code>currency</code> | R | - | R | - | R | - | R
<code>total</code> | R | - | R | - | R | - | R 
<code>taxes</code> | O | - | O | - | O | - | O
<code>details</code> | O | - | O | - | O | - | O
allowPartial | - | - | R | - | - | - | -
shipping | O | - | O | - | O | - | O
<code>documentType</code> | O | - | O | - | O | - | O
<code>document</code> | O | - | O | - | O | - | O 
<code>name</code> | O | - | O | - | O | - | O
<code>surname</code> | O | - | O | - | O | - | O
<code>company</code> | O | - | O | - | O | - | O
<code>email</code> | O | - | O | - | O | - | O
<code>address</code> | O | - | O | - | O | - | O
<code>mobile</code> | O | - | O | - | O | - | O
items | O | - | O | - | O | - | O
fields | O | - | O | - | O | - | O
recurring | - | - | - | - | R | - | O
<code>periodicity</code> | - | - | - | - | R | - | O
<code>interval</code> | - | - | - | - | R | - | O
<code>nextPayment</code> | - | - | - | - | R | - | O
<code>maxPeriods</code> | - | - | - | - | R | - | O
<code>dueDate</code> | - | - | - | - | R | - | O
<code>notificationUrl</code> | - | - | - | - | R | - | O
**subscription** | - | R | - | - | - | - | -
reference | - | R | - | - | - | - | - 
description | - | R | - | - | - | - | - 
fields | - | O | - | - | - | - | -
**fields** | O | O | O | - | O | - | O
keyword | O | O | O | - | O | - | O
value | O | O | O | - | O | - | O
displayOn | O | O | O | - | O | - | O
**paymentMethod** | O | O | O | - | O | - | -
**expiration** | R | R | R | - | R | - | -
**returnUrl** | R | R | R | - | R | - | -
**cancelUrl** | O | O | O | - | O | - | -
**ipAddress** | R | R | R | - | R | - | -
**userAgent** | R | R | R | - | R | - | -
**requestId** | - | - | - | R | - | R | -
**Instrument** | - | - | - | - | - | - | R
     token | - | - | - | - | - | - | R
     <code>token</code> | - | - | - | - | - | - | R*
    <code>subtoken</code> | - | - | - | - | - | - | R*
    <code>instalments</code> | - | - | - | - | - | - | O
    <code>cvv</code> | - | - | - | - | - | - | O

### Tarjeta de pruebas

Franchise | Card number | Comportamiento
----------|-------------|---------------
Visa | 4005580000000040 | Rechaza
Visa | 4007000000027 | Aprueba
Visa | 4111111111111111 | Aprueba
Visa | 4212121212121214 | Deja la operación pendiente como modo de captura, la operación debe ser autorizada o cancelada en el panel de Place to Pay  o de otra manera por operaciones de VOID o SETTLE.
Visa | 4666666666666669 | Este toma 5 minutos para autorizar. La idea es simular un tiempo de espera en su autorización. Así que el servicio de consumo fallará por el tiempo, lo que obligará al uso del Webservice para verificar cuando la operación completa su proceso. Tenga en cuenta los tiempos de consumo de Webservice.
Visa | 36545407032780 | Deja la operación en estado Manual (se debe aprobar o rechazar desde la consola)
MasterCard | 5424000000000015 | Aprueba
MasterCard Credencial (BCO) | 540625 10 00 00 00 08 | Aprueba
AmericanExpress | 370000000000002 | Aprueba
Diners | 36018623456787 | Aprueba
BBVA Club Campestre | 8130010000000000 | Aprueba
Visa Electron (Debit card) | 4027390000000006 | Aprueba
Visa Electron (Debit card) | 4215440000000001 | Rechaza
Codensa | 5907120000000009 | Rechaza
Tarjeta RIS | 6372000000000007 | Rechaza

<aside class="notice">
Los número de tarjetas solo tienen dicho comportamiento en el ambiente de pruebas.
</aside>

### Métodos de Pago Códigos.
    Esta es una lista de las franquicias disponibles en redirección.

País | Franquicia  | Método de pago | Descripción
-----|-------------|----------------|------------
Colombia | AC_WU | ath | Wester Union
 | AV_AV | ath | Banco AV Villas Recaudos
 |AV_BB | ath | Banco de Bogotá Recaudos
 | AV_BO | ath | Banco de Occidente Recaudos
 | AV_BP | ath | Banco popular Recaudos
 | BBVAC | ath |
 | BP_AM | amex |
 | BP_DN | diners |
 | BP_DS | discover |
 | BP_EL | elo |
 | BP_MC | master | 
 | BP_VS | visa | 
 | CAFAM | cafam | 
 | CDNSA | codensa | Codensa
 | CMRFB | falabella | 
 | CR_AM | amex | American Express
 | CR_CR | credencial | Credencial Banco de Occidente
 | CR_DN | diners | Diners Club
 | CR_VE | visa_electron | Visa Electron
 | CR_VS | visa | Visa
 | DBTAC | debito | Registro cuentas débito
 | DF_DN | diners | 
 | DF_DS | discover | 
 | DF_MC | master | 
 | DF_VS | visa | 
 | DISCO | discover | Disvover
 | DVVND | ath | 
 | EFCTY | efecty | Efecty
 | ENPCT | cooperativa | 
 | GNOFC | gana |
 | GNPIN | gana | GanaPIN
 | GNRIS | ris | Tarjeta RIS
 | ID_DN | diners | 
 | ID_DS | discover | 
 | ID_MC | master | 
 | ID_VS | visa | 
 | MSTRP | masterpass | Masterpass
 | PBBVA | ath | 
 | PINVL | Pin Valida | 
 | PYPAL | PayPal | PayPal
 | RM_MC | master | MasterCard
 | SOMOS | somos |
 | SB_VS | visa |
 | SFPAY | safetypay | Safety Pay
 | SPGRS | supergiros | 
 | T1_BC | ath | Bancolombia Recaudos
 | T1_CV | ath | 
 | TY_AK | Alkosto | Alkosto
 | TY_EX | Exito | Tarjeta Éxito
 | VISAC | visa_checkout | Visa Checkout
 | V_VBV | Verified by visa | 
 | _ATH_ | ath | Corresponsales bancarios Grupo Aval
 | _PPD_ | ppd | Débito pre-autorizado (PPD)
 | _PSE_ | pse | Débito a cuentas corrientes y ahorros (PSE)
Ecuador  | ID_VS | Visa |
         | ID_MC | Mastercard |
         | ID_DN | Diners |
         | ID_DS | Discover |

### Tipos de documento
Ésta es una lista de los tipos de documentos aceptados por redirección.

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