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

Franquicia | Número | Comportamiento
-----------|--------|----------------
Diners | 36545400000008 | Aprueba
Diners | 36545400000248 | Rechaza
Diners | 36545400000438 | Se tarda 5 minutos en aprobar
Diners | 36545400000701 | Deja la transacción en estado pendiente y al resolverse se aprueba
Diners | 36545407030701 | Deja la transacción en estado pendiente y al resolverse se rechaza
Visa | 4110770010002837 | Aprueba
Visa | 4111111111111111 | Aprueba
Visa | 4012888888881881 | Aprueba
Visa | 4005580000000040 | Rechaza
Visa | 4666666666666669 | Se tarda 5 minutos en aprobar
Visa | 36545407032780 | Deja la operación en estado Manual (se debe aprobar o rechazar desde la consola)

### Métodos de Pago Códigos.
    Esta es una lista de las franquicias disponibles en redirección.

País | Franquicia  | Método de pago
-----|--------|---------------
Colombia | AC_WU | ath
 | AV_AV | ath
 |AV_BB | ath
 | AV_BO | ath
 | AV_BP | ath
 | BBVAC | ath
 | BP_AM | amex
 | BP_DN | diners
 | BP_DS | discover
 | BP_EL | elo
 | BP_MC | master
 | BP_VS | visa
 | CAFAM | cafam
 | CDNSA | codensa
 | CMRFB | falabella
 | CR_AM | amex
 | CR_CR | credencial
 | CR_DN | diners
 | CR_VE | visa_electron
 | CR_VS | visa
 | DBTAC | debito
 | DF_DN | diners
 | DF_DS | discover
 | DF_MC | master
 | DF_VS | visa
 | DISCO | discover
 | DVVND | ath
 | EFCTY | efecty
 | ENPCT | cooperativa
 | GNOFC | gana
 | GNPIN | gana
 | GNRIS | ris
 | ID_DN | diners
 | ID_DS | discover
 | ID_MC | master
 | ID_VS | visa
 | MSTRP | masterpass
 | PBBVA | ath
 | PINVL | Pin Valida
 | PYPAL | PayPal
 | RM_MC | master
 | SOMOS | somos
 | SB_VS | visa
 | SFPAY | safetypay
 | SPGRS | supergiros
 | T1_BC | ath
 | T1_CV | ath
 | TY_AK | exito
 | TY_EX | alkosto
 | VISAC | visa_checkout
 | V_VBV | Verified by visa
 | _ATH_ | ath
 | _PPD_ | ppd
 | _PSE_ | pse
Ecuador  | ID_VS | Visa
         | ID_MC | Mastercard
         | ID_DN | Diners
         | ID_DS | Discover

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