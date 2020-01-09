# Conexión medios de pago API

Este Webservice y su arquitectura permiten que cualquier aplicativo sin importar el lenguaje de desarrollo, puedan beneficiarse de un conjunto de reglas de negocio y servicios, debido al cumplimiento de estándares SOAP, WSDL y XML.

Para hacer uso de los Webservice, debe contar con un lenguaje de programación que pueda consumirlos, algunos lenguajes tienen mejor soporte que otros desde el punto de vista de facilidad de acceso; sin embargo, si no se cuenta con un lenguaje que encapsule la comunicación, siempre se puede hacer incluso a nivel de sockets usando el protocolo HTTP conjuntamente con la especificación SOAP.

Para aquellos lenguajes que tienen un buen soporte lo más aconsejable es ir a través del WSDL para el Webservice.

La integración directa al medio de pago en general NO es recomendada, pues dificulta otros servicios que pueden estar implementados en la interfaz de Placetopay, al tiempo que hace que cada nuevo medio de pago liberado requiera un nuevo desarrollo por parte del comercio. Así que solo tener en cuenta esta integración si:

* Completa flujo de transacción no disponible por redirección.
* Usa el servicio a modo de marca blanca.

**¿Qué obligaciones tengo al usar esta integración?**

Ten en cuenta que al usar este tipo de integración, se requiere de certificación y los tiempos de implementación pueden ser mucho más altos que cuando se integra por [WebCheckOut](#webcheckout).

**Captura de la información**

Al consumir el servicio de creación de la transacción, tu aplicación debe proveer información sobre el pagador, el comprador y la transacción. Si corresponden a la misma persona, solamente deberás especificar la información del pagador

Para proveer estos datos, tu aplicación deberá capturarlos directamente en el proceso o de alguna fuente de información previamente habilitada.

<aside class="notice">
Recomendamos tener un certificado digital SSL para tu sitio.
</aside>