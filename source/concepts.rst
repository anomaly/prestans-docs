========
Concepts
========

Before you begin building REST services with prestans, it's important that you understand it's key concepts and the life cycle of a REST API call.

From the outset prestans will handle all trivial cases of validation, non matching URLs, authentication and convey an appropriate error message to the client. The following is the life cycle of a prestans API request:

* URL Routers checks for a handler mapping
* Router checks to see if the handler implements the requested method (GET, PUT, POST, PATCH, DELETE)
* If required checks to see if the user is allowed to access
* Unserializes input from the client
* Runs validation on URL parameters, body models and makes them available via the request object
* Runs pre-hook methods for handlers (use this for establishing DB connections, environment setup)
* **Runs your handler implementation, where you place your API logic**
* Runs post-hook methods for handlers (use this to perform your tear down)
* Serializes your output

Serializers
===========

Serializers are pluggable components that pack and unpack REST data in a seriazable format. For performance reasons most of them are wrappers on existing Python libraries, there's nothing stopping you from implementing one purely in Python.

You should never have to serialize or unserialize data when writing prestans apps, this is soley a job for the serializers. If serialization or unserialization fails, exceptions are raised and prestans sends out a canned error message to the client.

* JSON
* YAML

.. note:: We are working on XML support, and might settle for AtomPub.

REST Application
================

REST Application is our router, it's an instance of REST Application that maps a URL to a handler. It's also responsible managing the API call lifecycle and humanising error messages for the client.

REST Application can not be used directly, you must use a sub class that's been paired with a Serializer. Out of the box prestans provides the following REST Application routers:

* JSONRESTApplication
* YAMLRESTApplication

It's possible to write your serializer and REST Application, you should only have to do this if you want to use a format not supported by prestans.

Handlers
========

Handlers are end points where an API request URL maps to. It's here that your business logic should live and how prestans knows where to hand over to your code. A handler maps to a URL pattern. Handlers should define an instance method for each HTTP method that you want to support.

Regex matched patterns are passed to your handler functions as parameters. Handlers can choose to use RequestParsers to validate incoming requests.

.. _models:

Models
======

Models are a set of rules that can be used by a prestans parser to validate the body of the request. Models are also use to validate and even auto generate responses from persistent data models.

prestans Models descriptions are quite similar to Django or Google AppEngine models.

Attributes can be of the following types, these are in accordance with popular serialization formats for REST APIs:

* String
* Integer
* Float
* Boolean
* Date Time
* Arrays

Each attribute provides a set rules configured by you, that prestans uses to validate incoming and outgoing data.

Parsers
=======

Parameter Sets
==============

Are groups of prestans Data Types that can be accepted in 

Data Adapters
=============

Data Adapters are a set of handy tools that allow you to quickly turn persistent data objects into instances of your REST models. prestans allows serialization of prestans managed Data Types, see :ref:`models`. This is a backend specific feature.

You can map persistent models against prestans Models using a registry and ask prestans to perform the translation to construct your response.

Providers
=========

prestans was designed ground up to live along side other Python Web development frameworks, and work under any WSGI compliant environment. This presents us with a challenge of fitting into services that may already be in use by your application or environment.

Providers are wrappers that present prestans with an standardised way to talk to these environment specific services. The provider implements specific code to return the status that prestans expects.

We provide extensive documentation on writing your own providers for environments we don't support out of the box.

These services include:

* Authentication
* Caching

