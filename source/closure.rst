==============================================
Google Closure Library Extensions (incomplete)
==============================================

`Google Closure <https://developers.google.com/closure/library/>`_ is a set of JavaScript tools, that Google uses to build many of their core products. It provides:

* A `JavaScript Optimizer <https://developers.google.com/closure/compiler>`_ to build a distributable version of your application
* A comprehensive `JavaScript library <https://developers.google.com/closure/library>`_
* A `templating system <https://developers.google.com/closure/templates>`_ for JavaScript
* A `JavaScript style checker <https://developers.google.com/closure/utilities>`_ and style fixer
* An `enhanced stylesheet language <http://code.google.com/p/closure-stylesheets/>`_ that works with the optimizer to minifiy CSS.

Each one of these components is agnostic of the other. Closure is at the heart of building products with prestans.

prestans provides a number of extensions to Closure Library, that ease and automate building rich JavaScript clients that consume your prestans API. Our current line up includes:

* REST Client, provides a pattern to create Xhr requests, manages the lifecycle and parsers responses
* Types API, a client side replica of the prestans server types package assisting with parsing responses
* Code generation tools to quickly produce client side stubs from your REST application models

It's expected that you will use the Google Closure `dependency manager <https://developers.google.com/closure/library/docs/introduction>`_ to load the prestans namespaces.

Types API
=========

The Types API is a client side implementation of the prestans types API found on the server side. It assists in directly translating validation rules for Web based clients consuming REST services defined using prestans. Later in this chapter we discuss a set of tools that cut out the laborious job of creating client side stubs of your prestans models.

* ``String``, wraps a string
* ``Integer``, wraps a number
* ``Float``, wraps a number
* ``Boolean``, wraps a boolean
* ``DateTime``, wraps a `goog.date.DateTime <http://closure-library.googlecode.com/svn/docs/class_goog_date_DateTime.html>`_ and includes format configuration from the server side definition.
* ``Array``, extends `goog.iter.Iterator <http://closure-library.googlecode.com/svn/docs/class_goog_iter_Iterator.html>`_ enables you to use ``goog.iter.forEach``, we wrap most of the useful methods provided by Closure iterables.
* ``Model``, wraps JavaScript ``object``
* ``Filter`` is an configurable filter that you can pass with API calls, this translates back into attribute strings, discussed in :doc:`validation`.

Array
-----

``prestans.types.Array`` is iterable, it extends ``goog.iter.Iterator``. 

REST Client
===========

prestans contains a ready made REST Client to allow you to easily make requests and unpack responses from a prestans enabled server API. Our client implementation is specific to be used with Google Closure and only speaks `JSON`.

Client
------

``prestans.rest.Client``

* ``baseUrl``
* ``opt_numRetries`` set to 0 by default, causing requests never to be retired


Instance methods:

* ``abortAllPendingRequests``
* ``makeRequest`` request, callbackSuccessMethod, callbackFailureMethod, opt_abortPreviousRequests

Events
^^^^^^

* ``prestans.rest.json.Client.EventType.RESPONSE``
* ``prestans.rest.json.Client.EventType.FAILURE``

Request
-------

Requests ``prestans.rest.Request``

``prestans.rest.json.Request``

* ``identifier`` unique string identifier for this request type
* ``cancelable`` boolean value to determine if this request can be canceled
* ``httpMethod`` a ``prestans.net.HttpMethod`` constant
* ``parameters`` an array of key value pairs send as part of the URL
* ``requestFilter`` optional instance of ``prestans.types.Filter``
* ``requestModel`` optional instance of ``prestans.types.Model``, this will be used to parse the response message body
* ``responseFilter`` optional instance of ``prestans.types.Filter``, used to ignore fields in the response
* ``responseModel`` Used to unpack the returned response
* ``arrayElementTemplate`` Used if response model is an array
* ``responseModelElementTemplates`` 
* ``urlFormat`` sprintf like string used internally with `goog.string.format <http://closure-library.googlecode.com/svn/docs/namespace_goog_string.html>`_
* ``urlArgs`` a JavaScript array of parameters used with ``urlFormat``

``prestans.net.HttpMethod`` encapsulate HTTP verbs as constants, currently supported verbs are:

* ``prestans.net.HttpMethod.GET``
* ``prestans.net.HttpMethod.PUT``
* ``prestans.net.HttpMethod.POST``
* ``prestans.net.HttpMethod.DELETE``
* ``prestans.net.HttpMethod.PATCH``

Response
--------

* ``requestIdentifier`` The string identifier for the request type,
* ``statusCode`` HTTP status code,
* ``responseModel`` Class used to unpack response body,
* ``arrayElementTemplate`` prestans.types.Model,
* ``responseModelElementTemplates``
* ``responseBody`` JSON Object (Optional)

Closure Unit Tests
==================

Tools
======

preplate
--------