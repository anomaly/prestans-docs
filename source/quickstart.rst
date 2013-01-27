===============
Getting Started
===============

prestans is a WSGI compliant REST server framework, best suited for use with applications that nearly their entire interface using JavaScript (using frameworks like `Google Closure <https://developers.google.com/closure/>`_) or a bespoke Mobile client. Although prestans is a standalone framework, it provides hooks (called Providers) to integrate with your application's authentication, caching and other such core services.

We have battle tested prestans under Apache (using mod_wsgi) and Google's AppEngine platform.

Code samples used throughout our documentation is available as a `Google AppEngine project <https://code.google.com/p/prestans-demo/>`_, we highly recommend you grab a copy so you can see how it all fits in.

.. note:: You will require a copy of `Google's AppEngine Python SDK <https://developers.google.com/appengine/downloads>`_ (v1.7.0+) to run the sample project.

A downloadable release is available for prestans demo, but we are consistentantly working on it, so you might want to check out the Subversion repository.

Our philosophy is **"take as much or as little of the project as you like"**, prestans was designed ground up to sit nicely along side other Python frameworks. Needless to say that a dynamic language such as Python lends itself extremely well to writing frameworks such as prestans, and highly scaleable Web applications.

And incase you are still wondering prestans is a latin word meaning *"excellent, distinguished, imminent".*

prestans is distributed under the terms and conditions of the New BSD license and is hosted on Google Code.

Features
========

* Validation or incoming and outgoing using strongly defined Models
* Pluggable architecture allowing prestans to plug into any authentication, caching and serialization requirements.
* A custom URL dispatcher that allows you to re-use handlers for multiple output formats.
* Data Adapters, that allows you to translate persistent objects into REST resources, with a single line of code.
* Validation of URL parameters using strong defined Parameter Sets.
* Dynamically filtering response fields when writing responses.

We also maintain a set of tools that leverages prestans's ``Model`` definition schema to generate boiler plate client side parsing of REST resources.

Installation
============

There are two ways you can install prestans, one using Python's distutils and the other is by simple including it with your project's source. This is pariticularly handy if you are using environments like Google's AppEngine.

Making prestans available system wide, download a copy of the latest release, unpack the tarball and run setup.py::

    $ cd prestans-1.0
    $ python setup.py install

Ensure that the version of Python you are installing prestans for is the version used by your Web server.

Things to consider when distributing prestans with your application:

* Make sure download target a particular release of prestans, it is not recommended to use the bleeding edge.
* If you prefer reference prestans as a Subversion external, ensure you use reference one of the ``tags``, it is not recommended to reference ``trunk``

Concepts
========

Before you begin building REST services with prestans, it's important that you understand it's key concepts.

Serializers
-----------

Serializers are pluggable components that pack and unpack REST data in a seriazable format. For performance reasons most of them are wrappers on existing Python libraries, there's nothing stopping you from implementing one purely in Python.

You should never have to serialize or unserialize data when writing prestans apps, this is soley a job for the serializers. If serialization or unserialization fails, exceptions are raised and prestans sends out a canned error message to the client.

* JSON
* YAML

.. note:: We are working on XML support, and might settle for AtomPub.

REST Application
----------------

REST Application is our router, an instance of REST Application is responsible for mapping URLs to handlers. It's also responsible managing the API call lifecycle and humanising error messages for the client.

REST Application can not be used directly, you must use a sub class that's been paired with a Serializer. Out of the box prestans provides the following REST Application routers:

* ``prestans.rest.JSONRESTApplication``
* ``prestans.rest.YAMLRESTApplication``

It's possible to write your serializer and REST Application, you should only have to do this if you want to use a format not supported by prestans.

Handlers
--------

Handlers are end points where an API request URL maps to. It's here that your business logic should live and how prestans knows where to hand over to your code. A handler maps to a URL pattern. Handlers should define an instance method for each HTTP method that you want to support.

Regex matched patterns are passed to your handler functions as parameters. Handlers can choose to use RequestParsers to validate incoming requests.

.. _models:

Models
------

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

Request Parsers
---------------

Request Parsers allow you to define a set of rules that a request handler can use to validate incoming and outgoing data. Rules are define per HTTP method each handler corresponds supports and allows you to:

* validate sets of parmaeters in the URL
* the body of the request (for POST, PUT, PATCH and DELETE methods) by defining :ref:`models`
* a response attribute list template which allows clients to request partially formed responses, the template directly corresponds to the definition of the handler's response format
* a definition of acceptable partially formed requests (based on models)

Complimentary to Request Parsers are ``ParameterSet`` which allow you defined patterns of acceptable groups of parameters in the URL and ``AttributeFilter`` which allow you to make exceptions to the rules defined by Models.

Data Adapters
-------------

Data Adapters are a set of extensions that allow you to quickly turn persistent data objects into instances of your REST models. prestans allows serialization of prestans managed Data Types, see :ref:`models`. Data Adapters are backend specific (we currently support SQLAlchemy, AppEngine NDB).

These Adapters function map persistent models against prestans Models using a registry, allowing prestans to perform the translation to construct your  REST handler's response.

Providers
---------

prestans was designed ground up to live along side other Python Web development frameworks, and work under any WSGI compliant environment. This presents us with a challenge of fitting into services that may already be in use by your application or environment.

Providers are wrappers that present prestans with an standardised way to talk to these environment specific services. The provider implements specific code to return the status that prestans expects.

We provide extensive documentation on writing your own providers for environments we don't support out of the box.

These services include:

* Authentication
* Caching

