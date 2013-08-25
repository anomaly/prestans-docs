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
