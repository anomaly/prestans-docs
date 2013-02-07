===========================
Routing & Handling Requests
===========================

First order of business is mapping URLs back to your code. prestans comes with an inbuilt router to help you achieve this, the router is paired with a serializer and manages the lifecycle of the REST API request. Currently prestans provides support for:

* JSON provided by ``prestans.rest.JSONRESTApplication``
* YAML provided by ``prestans.rest.YAMLRESTApplication``

We plan to support other formats as we need them. You can also write your own for formats you wish to support in your application. Read the section on :doc:`serializers` to learn more about how serailziers work and how you can write your own. Our examples assume the use of JSON as the serialization format.

Each ``RESTApplication`` sub class paired with a serialzier is used to route URLs to handlers.

.. warning:: Do not attempt to use an instance of ``prestans.rest.RESTApplication`` directly.

API Request Lifecycle
=====================

From the outset prestans will handle all trivial cases of validation, non matching URLs, authentication and convey an appropriate error message to the client. It's important that you understand the life cycle of a prestans API request, you can use predefined Exceptions to automatically convey appropriate status codes to the client:

* URL Routers checks for a handler mapping
* Router checks to see if the handler implements the requested method (GET, PUT, POST, PATCH, DELETE)
* If required checks to see if the user is allowed to access
* Unserializes input from the client
* Runs validation on URL parameters, body models and makes them available via the request object
* Runs pre-hook methods for handlers (use this for establishing DB connections, environment setup)
* **Runs your handler implementation, where you place your API logic**
* Runs post-hook methods for handlers (use this to perform your tear down)
* Serializes your output

Regex & URL design primer
=========================

URL patterns are described using Regular expression, this section provides a quick reference to handy regex patterns for writing REST services. If you are fulent Regex speaker, feel free to skip this chapter.

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

Defining your REST Application
==============================

You must use a ``RESTApplication`` subclass (one that's paired with a serializer) to create map URLs to REST Handlers. A REST application accepts the following optional parameters:

* ``url_map`` a list of regex to REST handler maps
* ``application_name`` optional name for your API, this will show up in the logs.
* ``debug`` set to ``True`` by default, turn this off in production. This status is made available as ``self.request.debug`` 

url_map a non-optional parameter, requires pairs of URL patterns and REST Handler end points. The following example accepts two numeric IDs which are passed on to the handlers::

        (r'/api/band/([0-9]+)/album/([0-9]+)/track', pdemo.rest.handlers.track.Collection)

prestans would map this URL to the ``Collection`` class defined in the package ``pdemo.rest.handlers.track``, if you were to define a GET method which returned all the tracks for a band's album, it would look like::

        class Collection(prestans.rest.RESTHandler):

            def get(self, band_id, album_id):
                ... return all tracks for band_id and album_id

If your handler does not support an particuar HTTP method for a URL, simply ignore implementing the appropriate method.  An application API definition would be a collection of these URL to Handler pairs. The following is an extract from our demo application:

.. code-block:: python

    import prestans.rest

    import pdemo.handlers
    import pdemo.rest.handlers.album
    import pdemo.rest.handlers.band
    import pdemo.rest.handlers.track

    api = prestans.rest.JSONRESTApplication(url_handler_map=[
        (r'/api/band', pdemo.rest.handlers.band.Collection),
        (r'/api/band/([0-9]+)', pdemo.rest.handlers.band.Entity),
        (r'/api/band/([0-9]+)/album', pdemo.rest.handlers.album.Collection),
        (r'/api/band/([0-9]+)/album/([0-9]+)/track', pdemo.rest.handlers.track.Collection)
    ], application_name="prestans-demo", debug=False)

Configuring your WSGI environment
---------------------------------

Your WSGI environment has to be made aware of your declared prestans application. A Google AppEngine, app.yaml entry would look like::

    - url: /api/.*
      script: entry.api
      # Where the package entry contains an attribute called api

a corresponding ``entry.py`` would look like::

    #!/usr/bin/env/python

    import prestans.rest
    ... along with other imports

    api = prestans.rest.JSONRESTApplication(url_handler_map=[
        ... rules go here
    ], application_name="prestans-demo", debug=False)


Under Apache with `mod_wsgi <http://modwsgi.googlecode.com>`_ it a .wsgi file would look like (note that mod_wsgi requires the application attribute in the entry .wsgi script, best described in their `Quick Configuration Guide <http://code.google.com/p/modwsgi/wiki/QuickConfigurationGuide>`_)::

    #!/usr/bin/env/python

    import prestans.rest
    ... along with other imports
    
    application = prestans.rest.JSONRESTApplication(url_handler_map=[
        ... rules go here
    ], application_name="prestans-demo", debug=False)


Accessing incoming parameters
=============================

Handlers can accept input as parts of the URL, or the query string, or in the acceptable serialized format in the body of the request (not available for GET requests):

* Patterns matched using Regular Expression are passed via as part of the function call. They are positionally passed. Default behaviour passes all parameters as strings.
* Query parameters are available as key / value pairs, accessiable in a handler as ``self.request.get('param_name')``
* Serializers attempt to parse the request body and make the end results available at ``self.request.pased_body``

prestans defines a rich API to parse Query Strings, parts of the URL and the raw serialized body:

* Router that calls each handler passing parts of the URL extracted using ``regex`` to the appropriate handler method. 
* Use of Parameter Sets to parse set of acceptable queries, so your handlder doesn't have to worry about if the parameters in the query string are acceptable.
* Use of :doc:`models` and defined types to parse the body of requests, once again releaving you of checking the validity of the body.

This is a signature feature of our framework, and we have dedicated an entire chapter to discuss :doc:`validation`.

Writing Responses
=================

Each handler method in your prestans REST application must return either a:

* Python serializable type, these include basic types are iterables
* Instances of ``prestans.types.DataType`` or subclasses

To write a response you must:

* Set a proper HTTP response code, by setting ``self.response.status_code`` to a constant in ``prestans.rest.STATUS``
* Populating the body of the response

By default the response is set to a dictionary. Remember that at the end of the REST request lifecycle the response data is sent to the serializer. If your handler is sending arbitary data back to the client, it's suggested you use a key / value scheme to form your response.

``prestans.rest.Response`` provides the ``set_body_attribute`` method, which takes a string key and seriliable value:

.. code-block:: python

    import prestans.rest

    class AlbumEntityHandler(prestans.rest.RESTHandler):

        def get(self, band_id, album_id):

            # Set the handler status code to 200
            self.response.http_status = prestans.rest.STATUS.OK 

            # Add new attribute
            self.response.body.set_body_attribute("name", "Dark side of the moon")

prestans provides a well defined API to defined models for your REST API layer. These models are views on your persistent data and perform strong validation relfecting your business logic.

It's highly recommended to use :doc:`models` to form strongly validated responses. In addition prestans provides a set of :doc:`ext` that ease translation of persistent models to prestans REST models.

Using pre-defined exceptions
----------------------------

REST applications should use the breath of HTTP status codes to add meaning to the responses. prestans defines and handles a set of common expcetions that can be used by your application to send our standardised error responses. These ``Exception`` classes are paired with a status code and accept a string message as part of the constructor.

The string message is meant to make the error message more meaningful to the consumer of the API. Imagine the client wants to fetch an album for a band, it calls the album service with a ``band_id`` and an ``album_id``, if the album is not found or does not belong to the band, the service should throw return the status code of ``404`` Not Found with enough information that the client can act upon it.

It's not important to echo back values they sent as part of the request, as they should already have access to the original request.

A snippet that outlines this example would look as follows:

.. code-block:: python

    import prestans.rest

    class AlbumEntityHandler(prestans.rest.RESTHandler):

        def get(self, band_id, album_id):

            ... fetch the album that matches band_id and album_id

            # Raise an exception if the album was not found or didn't belong to the band
            if fetched_album is None or not fetched_album.band_id == int(band_id):
                raise prestans.rest.NotFoundException("Album")

            # Set the handler status code to 200
            self.response.http_status = prestans.rest.STATUS.OK 

            ... and return the album serialized in the appropriate format


The following are a list of exceptions provided by prestans along with their paired status code and suggestions for use cases:

+-------------------------------------------+---------------------------+------------------------------------------------------------------+
| Class                                     | HTTP status code          | Use cases                                                        |
+===========================================+===========================+==================================================================+
| prestans.rest.ServiceUnavailableException | 503 (Service Unavailable) | The REST service or a related backend service is unavailable     |
+-------------------------------------------+---------------------------+------------------------------------------------------------------+
| prestans.rest.BadRequestException         | 400 (Bad Request)         | Parameters sent as part of the request are not acceptable        |
+-------------------------------------------+---------------------------+------------------------------------------------------------------+
| prestans.rest.ConflictException           | 409 (Conflict)            |                                                                  |
+-------------------------------------------+---------------------------+------------------------------------------------------------------+
| prestans.rest.NotFoundException           | 404 (Not Found)           | The requested entity does not exists                             |
+-------------------------------------------+---------------------------+------------------------------------------------------------------+
| prestans.rest.UnauthorizedException       | 401 (Unauthorised)        | The request entity can not be accessed by the current client     |
+-------------------------------------------+---------------------------+------------------------------------------------------------------+
| prestans.rest.ForbiddenException          | 403 (Forbidden)           |                                                                  |
+-------------------------------------------+---------------------------+------------------------------------------------------------------+

It it obviously possible to use the other error codes by manually setting the handler's resposne code and body message.
