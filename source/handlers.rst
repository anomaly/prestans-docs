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

* /api/album/ - refers to a collection of type album
* /api/album/{id} - refers to a specific album entity

Notice no trailing slashes at the end of the entity URL. Collection URLs may or may not have a URL slash. The above patterns can would be represented in like Regex as: 

* /api/album/* - For collection of albums
* /api/album/([0-9]+) - For a specific album

If you have entities that exclusively belong to a parent object, e.g. Albums have Tracks, we suggest prefixing their URLs with a parent entity id. This will ensure your handler has access to the {id} of the parent object, easing operations like:

* Does referenced parent object exists?
* When creating a new child object, which parent object would you like to add it to? 
* Does the child belong to the intended parent (Works particularly well with ORM layers like SQLAlchemy)

A Regex example of these URL patterns would look like:

* /api/album/([0-9]+)/track/*
* /api/album/([0-9]+)/track/([0-9]+)

Defining your REST Application
------------------------------

Our documentation

* url_map
* application_name
* debug

Building your Response
======================

.. note:: Data Adapters help you quickly turn you persistent data to REST models instances.

