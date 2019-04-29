# Errores

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

El servicio de Web CheckOut retorna los Códigos de respuesta para la autenticación fallida en el atributo <code>message</code> del objeto <code>status</code>.


Código | Causa
---------- | -------
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

* **Error conectando al servicio con el mensaje**
<code>ERROR: javax.net.ssl.SSLHandshakeException: Remote host closed connection during handshake</code>
Tus servidores requieren TLSv1.2 para recibir la solicitud, debido a la norma PCI. Por favor, revisa el cifrado y el protocolo utilizado para conectar al servidor. Si usas Java, ten presente que solo las versiones después de la 8 tienen soporte completo.
