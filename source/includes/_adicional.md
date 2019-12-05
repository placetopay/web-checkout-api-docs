# Información adicional

## Campos usados por tipo de operación
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
surname O | O | O | - | O | - | O
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

## Métodos de Pago.
    Esta es una lista de las franquicias disponibles en redirección.

<<<<<<< HEAD
País | Código | Método de pago
=======
País | Código | Metodo de pago
>>>>>>> d23cf869ef50574b39cd5bca47b788efc5493434
-----|--------|---------------
Colombia | CR_VS | Visa
         | CR_CR | Credencial Banco de Occidente
         | CR_VE | Visa Electron
         | CR_DN | Diners Club
         | CR_AM | American Express
         | RM_MC | MasterCard
         | TY_EX | Tarjeta Éxito
         | TY_AK | Alkosto
         | _PSE_ | Débito a cuentas corrientes y ahorros (PSE)
         | SFPAY | Safety Pay
         | _ATH_ | Corresponsales bancarios Grupo Aval
         | AC_WU | Western Union
         | PYPAL | PayPal
         | T1_BC | Bancolombia Recaudos
         | AV_BO | Banco de Occidente Recaudos
         | AV_AV | Banco AV Villas Recaudos
         | AV_BB | Banco de Bogotá Recaudos
         | VISAC | Visa Checkout
         | GNPIN | GanaPIN
         | GNRIS | Tarjeta RIS
         | MSTRP | Masterpass
         | DBTAC | Registro cuentas débito
         | _PPD_ | Débito pre-autorizado (PPD)
         | CDNSA | Codensa
Ecuador  | ID_VS | Visa
         | ID_MC | Mastercard
         | ID_DN | Diners
         | ID_DS | Discover


## Tipos de documento
<<<<<<< HEAD
Ésta es una lista de los tipos de documentos aceptados por redirección.
=======
Esta es una lista de los tipos de docuemntos aceptados por redirección.
>>>>>>> d23cf869ef50574b39cd5bca47b788efc5493434

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