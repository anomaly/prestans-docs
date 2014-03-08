================
Framework Design
================

prestans is a developer-to-developer product. Before we start talking about how you can start harnessing it's features, we thought it might be a good idea to introduce you to some of the design decisions and patterns. This chapter highlights some of under the hood design decisions, it's written so you can gain an understanding of why prestans behaves the way it does.

It covers the use of HTTP headers to control features of prestans, and when prestans will decide to bail out processing a request and when it will choose to fail gracefully (still responding back to the requesting client).

If you trust that we have made all the right decisions and you'd rather start working on your prestans powered API, then head straight to :doc:`handlers`

Exceptions
==========

prestans raises two kinds of exceptions. It handles both of them in very different ways:

* The first are inbuilt Python exceptions e.g ``TypeError``, ``AssertionError``, prestans does not handle these exceptions, causing your API to fail process that particular request. This is because these are only raised if you have incorrectly configured your prestans handler, or more importantly your handler is not respecting it's own rules e.g objects returned by an end point don't match your parser configuration.

* The second are prestans defined exceptions. These are handled by prestans and the router will return a proper error response to the requesting client. These are raised if the client placed an incorrect request e.g An attribute wasn't the right length or type. These excpetions are defined in ``prestans.exceptions``, along with a guide for it's suggested use.

prestans defined exceptions can also be used by your handler to notify the client of trivial REST usecases e.g requested entity does not exists. We talk about these in :doc:`handlers`.

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