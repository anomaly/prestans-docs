======================
Understanding prestans
======================

This is 

* Serializers
* Router
* Handler
* Models
* Parsers
* Data Adapters
* Providers


Serializers
===========

* Textual
* Binary

prestans HTTP Headers
=====================

prestans uses several HTTP headers to evaluate requests. This section outline the rules in the prestans lifecycle.

Each request must send an ``Accept`` header for prestans to decide the resposne format. If the registered handler cannot respond in the requeted format prestans raises an ``UnsupportedVocabularyError`` exception inturn producing a ``501 Not Implemented`` response. All prestans APIs have a set of default formats all handlers accept, each end-point might accept additional formats.

If a request has send a body (e.g ``PUT``, ``POST``) you must send a ``Content-Type`` header to declare the format in use. If you do not send a ``Content-Type`` header prestans will attempt to use the default deserializer to deserialize the body. If the ``Content-Type`` is not supported by the API an ``UnsupportedContentTypeError``` exception is raised inturn producing a ``501 Not Implemented`` response.

Inbound headers
---------------

A client may send the following 

* ``Prestans-Version``
* ``Prestans-Response-Attribute-List``
* ``Prestans-Response-Minification``

Sent by the server:

* ``Prestans-Version``
* ``Prestans-Rewrite-Map``

Request lifecycle
==================

* Is the verb supported
* Is the requested mime type supported
* 
