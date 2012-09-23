===========================
Routing & Handling Requests
===========================

First order of business is mapping URLs back to your code. prestans comes with an inbuilt router to help you achieve this, the router is paired with a serializer and manages the lifecycle of the REST API request. Out of the box prestans provides support for:

* JSON
* YAML

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

Regex refresher
===============

URL patterns are described using Regular expression, this chapter is a quick overview on handy regex patterns. If you are fulent Regex feel free to skip this chapter.

Most URL patters either refer to collections or entities, consider the following URL scheme requirements:

* /api/album/ - refers to a collection of type album
* /api/album/{id} - refers to a specific album entity

Notice no trailing slashes at the end of the entity URL, this denotes an signle API

* /api/product/([0-9]+)
* /api/album/([0-9]+)/track/
* /api/album/([0-9]+)/track/([0-9]+)

Defining your REST Application
==============================

* url_map
* application_name
* debug

Building your Response
======================

.. note:: Data Adapters help you quickly turn you persistent data to REST models instances.

