## Errores

>Ejemplo atutenticación fallida:

```shell
{
    "status": {
        "status": "FAILED",
        "reason": 401,
        "message": "Authentication Failed 103",
        "date": "2019-03-06T17:08:36-05:00"
    }
}
```

El servicio de Web CheckOut retorna los códigos de respuesta para la autenticación fallida en el atributo <code>message</code> del objeto <code>status</code>.

Código | Causa
------ | ------
100 | UsernameToken no proporcionado (Encabezado de la autorización malformado).
101 | Identificador de sitio no existe ( Login incorrecto o no se encuentra en el ambiente).
102 | El hash de TranKey no coincide (Trankey incorrecto o mal formado).
103 | Fecha de la semilla mayor de 5 minutos.
104 | Sitio inactivo.
105 | Sitio expirado.
106 | Credenciales expiradas.
107 | Mala definición del UsernameToken (No cumple con el encabezado WSSE).
200 | Saltar el encabezado de autenticación SOAP.
10001 | Contacte a Soporte.

<aside class="warning">
Si el error retornado no corresponde a ninguna anteriormente descrito, por favor revise el endpoint de consumo usado.
</aside>.

### Errores frecuentes

* **Mensaje de error “Autenticación mal formada”** <br><br>
Se presenta cuando el sistema no detecta que se esté enviando login, tranKey, seed o nonce en la estructura auth enviada, también puede presentarse si se envían estos datos pero de manera incorrecta, es decir sin el parámetro content-type “application/json” de manera que el servidor interpreta la petición como texto en vez de un arreglo de datos.
Puedes validar esto haciendo la petición a la URL [https://dnetix.co/p2p/client](https://dnetix.co/p2p/client) y capturando la respuesta, es una especie de espejo de la petición que te permitirá comprobar los parámetros y el body del mensaje.

* **Error conectando al servicio con el mensaje** <code>ERROR: javax.net.ssl.SSLHandshakeException: Remote host closed connection during handshake</code> <br><br>
Tus servidores requieren TLSv1.2 para recibir la solicitud, debido a la norma PCI. Por favor, revisa el cifrado y el protocolo utilizado para conectar al servidor. Si usas Java, ten presente que solo las versiones después de la 8 tienen soporte completo.

* **SoapFault responde con el mensaje "Authentication Failed 103"** <br><br> 
En el proceso de autenticación nosotros revisamos el campo <code>Created</code>, este campo debe estar en el tiempo GMT o el tiempo local usando el tiempo de zona. Si obtienes esta respuesta, se debe a que tu tiempo no es preciso con el tiempo real. Nosotros solo permitimos 5 minutos de diferencia entre los tiempos.
Puedes usar NTP para mantener la precisión del reloj. 

* **Dando los mismos valores EXACTOS que en los ejemplos anteriores a la <code>BASE64(SHA1($Nonce + $Created . $tranKey))</code> estoy obteniendo un <code>password digest</code> diferente.** <br><br> 
Mantén en mente que BASE64 debería ser para el raw output  de la SHA1 y de acuerdo con todos los lenguajes de programación este puede ser requerido para configurar esta opción, por ejemplo.
En PHP base64_encode(sha1( … , true)) este parametro retornaria el <code>raw output</code> para el <code>sHA1 algorithm</code>