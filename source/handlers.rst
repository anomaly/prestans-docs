===========================
Routing & Handling Requests
===========================

First order of business is mapping URLs back to your code. prestans comes with an inbuilt router to help you achieve this, the router is paired with a serializer and manages the lifecycle of the REST API request. Currently prestans provides support for:

* JSON provided by ``prestans.rest.JSONRESTApplication``
* YAML provided by ``prestans.rest.YAMLRESTApplication``

We plan to support other formats as we need them. You can also write your own for formats you wish to support in your application. Read the section on :doc:`serializers` to learn more about how serailziers work and how you can write your own. Our examples assume the use of JSON as the serialization format.

Each ``RESTApplication`` sub class paired with a serialzier is used to route URLs to handlers.

.. note:: Do not attempt to use an instance of ``prestans.rest.RESTApplication`` directly.

API Request Lifecycle
---------------------

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
-------------------------

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
------------------------------

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
=================================

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


Building and Writing Responses
------------------------------

Each handler method in your prestans REST application must return either a:

* Python serializable type, these include basic types are iterables
* Instances of ``prestans.types.DataType`` or subclasses

Python types are passed directly to the serializer 

.. note:: Data Adapters help you quickly turn you persistent data to REST models instances.


Defining rules for your incoming data
-------------------------------------
