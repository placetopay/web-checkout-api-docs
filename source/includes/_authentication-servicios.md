## Authentication

> Ejemplo generación de fecha en formato [ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html):

```php

<?php
$seed = date('c');
?>
```

```csharp

using System;
using System.Globalization;

String seed = DateTime.UtcNow.ToString("s",
    System.Globalization.CultureInfo.InvariantCulture);

```

> Ejemplo generación Hash:

```php
<?php
    $hashString = sha1($seed 
    . $tranKey, false);
?>
```

```csharp
using System.Text;
using System.Security.Cryptography;

private string GetSHA1(string text)
{
    ASCIIEncoding UE = new ASCIIEncoding();
    byte[] hashValue;
    byte[] message = UE.GetBytes(text);
    SHA1 hashString = new SHA1CryptoServiceProvider();
    string hex = "";
    hashValue = hashString.ComputeHash(message);
    foreach (byte x in hashValue) {
        hex += String.Format("{0:x2}", x);
    }
    return hex;
}

string hashString = GetSHA1(seed
    + tranKey);
```

> Ejemplo de autenticación con la estructura <code>auth</code>:

```php
<?php
        $seed = date('c');
        $secretKey = '024h1IlD';
        $trankey = sha1($seed.$secretKey);
                           
        $servicio = 'https://test.placetopay.com/soap/pse/v11/?wsdl'; //url del servicio
    
    $auth = array(
        'login' => '6dd490faf9cb87a9862245da41170ff2',
        'tranKey' => $trankey,
        'seed' => $seed,
    );

    $arguments = array(
                        'auth' => $auth
                    );
    $client = new nusoap_client($servicio,array('trace' => true)); // En lugar de SoapClient se utiliza nusoap_client
?>
```

```shell
    POST /api/session
    {
    "auth": 
    {
        "login": "usuarioprueba",
        "tranKey": "T0O+x3gNlQUf0iBxEuenPvBPlWs=",
        "seed": "2019-04-25T18:17:23-04:00",
     }
    }
```

La autenticación a los servicios web debe ser enviada sobre la estructura <code>auth</code>, compuesto como se muestra a continuación:

ATRIBUTOS

| Nombre  | Tipo                    | Descripción                                                                                                                                                                                                                              |
|---------|-------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| login   | string                  | Identificador del sitio.                                                                                                                                                                                                                 |
| tranKey | string                  | Llave transaccional, para el consumo del API: <code>SHA-1(seed + trankey)</code>.                                                                                                                                                        |
| seed    | string                  | Fecha actual, la cual se genera en formato [ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html). El sistema verificará que la petición no haya expirado, por ello es fundamental una sincronización del reloj del servidor |
| additional   | [Attribute](#attribute) | Datos adicionales a la estructura de autenticación.                                                                                                                                                                                      |


<aside class="notice">
El login y el trankey son suministrados por Placetopay.
</aside>

<aside class="notice">
El Hash es el que se remite como tranKey en la estructura de autenticación.
</aside>

<aside class="notice">
La estructura de autenticación debe de ser enviada en cada petición realizada al servicio.
</aside>