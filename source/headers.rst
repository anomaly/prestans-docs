HTTP Header Reference
=====================

HTTP requests must contain all required information

HTTP headers are components of the message header for both HTTP requests and responses. They define the rules (e.g preferred content, cookies, software version) for each HTTP requests. These assist the server to best respond to the request and ensures that the client is able to consume it properly.

Headers can be specific to an HTTP Verb. prestans a combination of standard HTTP and a few custom headers to allow the requesting client to control the behavior of the API.

prestans provides a `client side <https://github.com/prestans/prestans-client/>`_ add-ons for `Google Closure <https://developers.google.com/closure/library/>`_ which supports the use of our HTTP rules.

Custom headers
--------------

prestans allows requesting APIs

Inbound headers:

* ``Prestans-Version``
* ``Prestans-Response-Attribute-List``
* ``Prestans-Minification``

Outbound headers:

* ``Prestans-Version``


Content Negotiation
-------------------

Each API request uses the following standard HTTP headers to negotatie the content type of the request and response payload:

* ``Accept`` - which tells the server formats that the client is willing to format, prestans compares this to what the current API support, or sends back an error message using the default serializer.
* ``Content-Type`` - which tells the server what format the request body is; this is only relevant for certain HTTP Verbs. If the prestans API does not support the format it sends back an error message using the default serializer.

.. note:: If you are new to REST, we highly recommend watching `Michael Mahemoff <http://mahemoff.com>`_'s `Web Directions Code 2013 <http://code13.webdirections.org>`_ presentation on REST, `What every developer should know about REST <https://www.youtube.com/watch?v=2yAQ-yLq5eI>`_.

Both these headers are part of the HTTP standards, the handler section in this chapter discusses how you pair up serializers and deserializers to these headers.


Request Sample
--------------

Assuming that we are writing an API that primarily speaks JSON, 