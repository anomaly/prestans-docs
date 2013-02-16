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

* REST Client, provides a pattern to create Xhr requests, manages the life cycle and parsers responses, also supports Attribute Fitlers.
* Types API, a client side replica of the prestans server types package assisting with parsing responses.
* Code generation tools to quickly produce client side stubs from your REST application models.

It's expected that you will use the Google Closure `dependency manager <https://developers.google.com/closure/library/docs/introduction>`_ to load the prestans namespaces.

.. note:: It's assumed that you are familiar with developing applications with Google Closure tools.

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

The client has three important parts:

* Request Manager provided by ``prestans.rest.json.Client``, this queues, manages, cancels requests and is resposible for firing callbacks on success and failure. Your application lodges all API call requests with an instance of ``prestans.rest.json.Client``. It's designed to be shared by your entire application.
* Request provided by ``prestans.rest.json.Request`` is a formalised request that can be passed to a Request Manager. The Request constructor accepts a JSON payload with configuration information, this includs partial URL schemes, parameters, optionaly a body and a format for the response. The Request Manager uses the responses format to parse the server response.
* Response provided by ``prestans.rest.json.Response`` encapsulates a server response. It also contains a parsed copy of the server response expressed using prestans types.

The general idea is:

* To maintain a globally accessible Request Manager 
* Formally define each Xhr operation as a Request object 
* The Request Manager handles the life cycle of a Xhr call and call an endpoint in your application on success or failure
* Both these callbacks are provided an instance of ``Response`` containing the appropriate available information

Request Manager
---------------

First step is to create a request manager by instantiating ``prestans.rest.json.Client``, it takes the following parameters:

* ``baseUrl``, to be consistent with the single point of origin constraint, we assume that all your API calls are prefixed with something like ``/api``. If you provide a base URL all your requests should provide URLs relative to the base. This also makes for eased maintenance in case you rearrange your application URLs.
* ``opt_numRetries`` set to 0 by default, causing requests never to be retried. Xhr implementations are capable of retrying to reach the server in case of failure.

There's a fair chance that your application might launch simultaneous Xhr requests, it's also likely that you would want to cancel some requests on events e.g as the user clicks around names of artists to get a list of their albums, you want to cancel any previously unfinished calls if the user has clicked on another artist name.

Our request manager can work this, this is done by using a shared instance of the request manager across your application. The following code sample demonstrates how you might maintain a global Request Manager instance:

.. code-block:: javascript

    goog.provide('pdemo');
    goog.require('prestans.rest.json.Client');

    pdemo.GLOBALS = {
        API_CLIENT: new prestans.rest.json.Client("/api", 0)
    };

Then use the ``makeRequest`` method on the Request Manager instance to dispatch API calls, it requires the following parameters:

* ``request`` is a ``prestans.rest.json.Request`` object.
* ``callbackSuccessMethod`` which is a reference to a function the Request Manager calls if the API call succeeds, the method will be passed a response object. Ensure you use ``goog.bind`` to bind your function to your namespace.   
* ``callbackFailureMethod`` optional reference to a function the Request Manager calls if the API call fails, this method will be passed a response object with failure information. 
* ``opt_abortPreviousRequests``, asks the Request Manager to cancel all pending requests.

.. code-block:: javascript

    # Assume you have a request object
    pdemo.GLOBALS.API_CLIENT.makeRequest(
        request,
        goog.bind(this.successCallback_, this),
        goog.bind(this.failureCallback_, this),
        false
    );

.. note:: Request objects tell the manager if they are willing to be aborted, this is configurable per request lodged with the manager.

The second method the Request Manager provides is ``abortAllPendingRequests``, this accepts no parameters and is responsible for aborting any currently queued connections. The failure callback is not fired when requests are aborted.

Events
^^^^^^

The Request Manager raises the following events. These come in handy if your application requires global UI interactions e.g a Modal popup if network communication fails, or notification messages on success.

* ``prestans.rest.json.Client.EventType.RESPONSE``, raised when a round trip succeeds, this would be raised even if your API raised an error code, e.g Bad Request or Service Unavailable.
* ``prestans.rest.json.Client.EventType.FAILURE`` raised if a round trip fails.

Example of using Closure's EventHandler to listen to these events:

.. code-block:: javascript

    goog.require('goog.events.EventHandler');

    # and somewhere in one of your functions
    this.eventHandler = new goog.events.EventHandler(this);
    this.eventHandler_.listen(pdemo.GLOBALS.API_CLIENT, prestans.rest.json.Client.EventType.FAILURE, this.handleFailure_);

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