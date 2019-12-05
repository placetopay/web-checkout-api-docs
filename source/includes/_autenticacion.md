## Autenticación

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
tranKey | Llave transaccional, que está formada por la siguiente operación: <code>Base64(SHA-1(nonce + seed + secretkey))</code>. El nonce dentro de la operación es el original, es decir, el que no está en Base64.
nonce | Valor aleatorio para cada solicitud codificado en Base64.
seed | Fecha actual, la cual se genera en formato [ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html).


<aside class="notice">
El login y el secretKey son suministrados por PlacetoPay
</aside>

<aside class="notice">
Esta estructura debe de ser enviada en cada petición realizada al servicio
</aside>

<aside class="warning">
El <code>seed</code> no puede tener diferencia de más de 5 minutos.
</aside>

### Ejemplos de autenticación

<a href="https://github.com/dnetix/redirection" target="_blank" style="text-decoration: none;">
 <img src="images/php.png" class="logo-lenguaje" alt="PHP">
</a>
<a href="https://gist.github.com/dnetix/f37de7864b4efb8249d30e476e379f0a" target="_blank" style="text-decoration: none;">
<img src="images/java.png" class="logo-lenguaje" alt="JAVA">
</a>
<a href="https://gist.github.com/dnetix/c18cc44861c5702d2b8ff2327b031c3e" target="_blank" style="text-decoration: none;">
<img src="images/c.png" class="logo-lenguaje" alt="C#">
</a>
<a href="https://gist.github.com/dnetix/fea3868afe915229c7d140967e4d8519" target="_blank" style="text-decoration: none;">
<img src="images/node.png" class="logo-lenguaje" alt="NodeJS">
</a>

Todas las solicitudes al web deben ser autenticadas con [WS Security UsernameToken Profile 1.1](https://www.oasis-open.org/committees/download.php/13392/wss-v1.1-spec-pr-UsernameTokenProfile-01.htm)

Usando <code>PasswordDigest (PasswordDigest = Base64 ( SHA-1 ( nonce + created + tranKey ) ))</code>
