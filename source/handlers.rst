===========================
Routing & Handling Requests
===========================


HTTP Headers
============

Inbound headers:

* ``Prestans-Version``
* ``Prestans-Response-Attribute-List``
* ``Prestans-Minification``

Outbound headers:

* ``Prestans-Version``

Content Negotiation
===================

Serializers & DeSerializers
---------------------------

Serializers and DeSerializers are pluggable prestans constructs that assist the framework in packing or unpacking data. Serialzier or deserializer handle content of a particular mime type and are generally wrappers to vocabularies already available in Python (although it possible to write a custom serializer entirely in Python). There are two types of serializers and deserializers:

* ``Textual`` - serialization formats that are text based (e.g JSON, XML, CSV) and are the ones that applications common use to communicate
* ``Binary`` - serialization formats that are binary (e.g PDF) and are mostly used as an output format.

prestans application may speak as many vocabularies as they wish; vocabularies can also be local to handlers (as opposed to applicaiton wide). You must also define a default format.

Each request must send an ``Accept`` header for prestans to decide the response format. If the registered handler cannot respond in the requeted format prestans raises an ``UnsupportedVocabularyError`` exception inturn producing a ``501 Not Implemented`` response. All prestans APIs have a set of default formats all handlers accept, each end-point might accept additional formats.

If a request has send a body (e.g ``PUT``, ``POST``) you must send a ``Content-Type`` header to declare the format in use. If you do not send a ``Content-Type`` header prestans will attempt to use the default deserializer to deserialize the body. If the ``Content-Type`` is not supported by the API an ``UnsupportedContentTypeError``` exception is raised inturn producing a ``501 Not Implemented`` response.

Routing Requests
================

.. code-block:: python

	import prestans.rest

	api = prestans.rest.RequestRouter(routes=[
	        (r'/([0-9]+)', DefaultHandler)
	    ], 
	    serializers=[prestans.serializer.JSON()],
	    default_serializer=prestans.serializer.JSON(),
	    deserializers=[prestans.deserializer.JSON()],
	    default_deserializer=prestans.deserializer.JSON(),
	    charset="utf-8",
	    application_name="music-db", 
	    logger=None,
	    debug=True)

* ``routes``
* ``serializers`` takes a list of serializer instances, if you ommit this paramter presntas will assign JSON as serializer to the API.
* ``default_serializer`` takes a serializer instance which it as use
* ``deserializers`` ``None``
* ``default_deserializer`` ``None``
* ``charset`` ``utf-8``
* ``application_name`` ``prestans``
* ``logger`` ``None``
* ``debug`` ``False``

* Configuring the router
* Debug mode
* Configuring default serializers
* Configuring logger, default logging configuration
* Adding routes


Describe the responsibilities of a request router


Handler 
=======

* Lifecycle of the handler

Registering additional serializers and deserializers


Logger
======


Raising Exceptions
==================


`PEP 008 <http://www.python.org/dev/peps/pep-0008/#exception-names>`_ says Exceptions that are errors should end with the Error suffix.

Configuration Exceptions are TypeErrors

If you hve 

Unsupported Vocabulary
----------------------


Data Validation Exceptions
--------------------------

Parser Exceptions
-----------------


Handler Exceptions
------------------

Not listed here
---------------