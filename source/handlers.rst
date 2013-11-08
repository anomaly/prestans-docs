===========================
Routing & Handling Requests
===========================

Web Server Gateway Interface or WSGI (`PEP-333 <http://www.python.org/dev/peps/pep-0333/>`_) is the glue between a Web Server and your Python Web application. The responding application simply has to be a Python `callable <http://docs.python.org/2/library/functions.html#callable>`_ (a python function or a class that implements the ``__call__`` method). Each *callable* is passed the Web server environment (much like CGI applications) and a ``start_response``. 

If you are not familiar with WSGI we recommend reading `Armin Ronarcher <http://lucumr.pocoo.org/>`_'s `introduction to WSGI <http://lucumr.pocoo.org/2007/5/21/getting-started-with-wsgi/>`_. We also have a great collection of :doc:`reference_material` on Python Web development.

WSGI interfaces will generally handover requests that match a URL pattern to the mapped WSGI callable. From the callable is responsible for dispatching the request to the appropriate handler based on part of the URL, HTTP verb, headers or any other property of an HTTP request or a combination properties. This middle ware code is refered to as a request router and prestans provides one of it's own.

prestans makes use of standard HTTP headers for content negotiation. In addition it uses a handful of custom headers that the client can use to control the prestans based API's behavior (features include Content Minification and Attribute Subsets for requests and responses). We'll first introduce you to the relevant HTTP and how it effects your API requests followed by how you can handle API requests in prestans.

Serializers & DeSerializers
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Serializers and DeSerializers are pluggable prestans constructs that assist the framework in packing or unpacking data. Serialzier or deserializer handle content of a particular mime type and are generally wrappers to vocabularies already available in Python (although it possible to write a custom serializer entirely in Python). There are two types of serializers and deserializers:

* ``Textual`` - serialization formats that are text based (e.g JSON, XML, CSV) and are the ones that applications common use to communicate
* ``Binary`` - serialization formats that are binary (e.g PDF) and are mostly used as an output format.

prestans application may speak as many vocabularies as they wish; vocabularies can also be local to handlers (as opposed to applicaiton wide). You must also define a default format.

Each request must send an ``Accept`` header for prestans to decide the response format. If the registered handler cannot respond in the requeted format prestans raises an ``UnsupportedVocabularyError`` exception inturn producing a ``501 Not Implemented`` response. All prestans APIs have a set of default formats all handlers accept, each end-point might accept additional formats.

If a request has send a body (e.g ``PUT``, ``POST``) you must send a ``Content-Type`` header to declare the format in use. If you do not send a ``Content-Type`` header prestans will attempt to use the default deserializer to deserialize the body. If the ``Content-Type`` is not supported by the API an ``UnsupportedContentTypeError``` exception is raised inturn producing a ``501 Not Implemented`` response.


Routing Requests
================

Regex & URL design primer
-------------------------

URL patterns are described using Regular expression, this section provides a quick reference to handy regex patterns for writing REST services. If you are fluent Regex speaker, feel free to skip this chapter.

Most URL patters either refer to collections or entities, consider the following URL scheme requirements:

* ``/api/album/`` - refers to a collection of type album
* ``/api/album/{id}`` - refers to a specific album entity

Notice no trailing slashes at the end of the entity URL. Collection URLs may or may not have a URL slash. The above patterns can would be represented in like Regex as: 

* ``/api/album/*`` - For collection of albums
* ``/api/album/([0-9]+)`` - For a specific album

If you have entities that exclusively belong to a parent object, e.g. Albums have Tracks, we suggest prefixing their URLs with a parent entity id. This will ensure your handler has access to the {id} of the parent object, easing operations like:

* Does referenced parent object exists?
* When creating a new child object, which parent object would you like to add it to? 
* Does the child belong to the intended parent (Works particularly well with ORM layers like SQLAlchemy)

A Regex example of these URL patterns would look like:

* ``/api/album/([0-9]+)/track/*``
* ``/api/album/([0-9]+)/track/([0-9]+)``

Using Request Router
--------------------

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
* ``serializers`` takes a list of serializer instances, if you omit this parameter prestans will assign JSON as serializer to the API.
* ``default_serializer`` takes a serializer instance which it as used if 
* ``deserializers`` ``None`` fill this out a little
* ``default_deserializer`` fill this out a little ``None``s 
* ``charset`` ``utf-8``
* ``application_name`` ``prestans``
* ``logger`` ``None``
* ``debug`` ``False``

Write about these things:

* Configuring the router
* Debug mode
* Configuring default serializers
* Configuring logger, default logging configuration
* Adding routes


Describe the responsibilities of a request router


Handling Requests
=================

* Lifecycle of the handler

Registering additional serializers and deserializers


Logger
------

Minifying Content
-----------------


Constructing Response
=====================

Raising Exceptions
------------------


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