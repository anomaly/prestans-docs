=================================
Google Closure Library Extensions
=================================

`Google Closure <https://developers.google.com/closure/library/>`_ is a set of JavaScript tools, that Google uses to build many of their core products. It provides:

* A `JavaScript Optimizer <https://developers.google.com/closure/compiler>`_ to build a distributable version of your application
* A comprehensive `JavaScript library <https://developers.google.com/closure/library>`_
* A `templating system <https://developers.google.com/closure/templates>`_ for JavaScript
* A `JavaScript style checker <https://developers.google.com/closure/utilities>`_ and style fixer
* An `enhanced stylesheet language <http://code.google.com/p/closure-stylesheets/>`_ that works with the optimizer to minifiy CSS.

Each one of these components is agnostic of the other. Closure is at the heart of building products with prestans.

Google Closure is unlike other JavaScript frameworks (e.g jQuery). An extremely central part of Closure tools is it's `compiler <https://developers.google.com/closure/compiler/>`_ (which is not just a minifier), the Closure development philosophy is to use the abstractions and components made available by Closure library and allow the compiler to optimise it for production.

.. note:: It's assumed that you are familiar with developing applications with Google Closure tools.

Prestans provides a number of extensions to Closure Library, that ease and automate building rich JavaScript clients that consume your Prestans API. Our current line up includes:

* REST Client, provides a pattern to create Xhr requests, manages the life cycle and parsers responses, also supports Attribute Fitlers.
* Types API, a client side replica of the Prestans server types package assisting with parsing responses.
* Code generation tools to quickly produce client side stubs from your REST application models.

It's expected that you will use the Google Closure `dependency manager <https://developers.google.com/closure/library/docs/introduction>`_ to load the Prestans namespaces.


Installation
============

Our client library follows the same development philosophy as Google Closure library, although we make available downloadable versions of the client library it's highly recommended that you reference our repository as an external source.

This allows you to keep up to date with our code base and benefit from the latest patches when you next compile.

Closure library does the same, and we ensure that we are leveraging off their latest developments.

.. note:: Code referenced in this section is available on `Github <http://github.com/prestans/prestans-client/>`_.

Unit testing
------------

Adjust the ``DEPSWRITER`` variable in the calcdeps.sh script and run it in the prestans-client directory.

.. code-block:: javascript

    cd prestans-client
    ./calcdeps.sh

To run these unit tests you will need to start Google Chrome with ``--allow-file-access-from-files`` parameter. Example on Mac OS X:

.. code-block:: bash
    
    /Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --allow-file-access-from-files

Extending JavaScript namespaces
===============================

:doc:`types` ensure the validity of data sent to and from the server. The application client should be as responsible validate data on the client side, ensuring that you never send an invalid request or you never accept an invalid response. Discussed later in this chapter are tools provided by Prestans that auto generate Closure library compatible versions of your server side Models and Attribute Filters, needless to say our JSON client works seamlessly with these auto generated Models and Filters.

Auto generated code is accompanied with the curse of losing local modifications (e.g adding a helper method or computed property) when you next run the auto generate process. 

Consider the following scenario, Prestans auto generates a Model class called ``User``, this uses the JavaScript namespace ``pdemo.data.model.User``, you now wish to write a function to say concatenate a user's first and last name. The obvious approach is to use ``goog.inherits`` to create a subclass of ``pdemo.data.model.User``. However for dynamic operations like parsing server responses maintaining the namespace is crucial.

Thanks to JavaScript's dynamic nature and Closure's excellent dependency management it's quite easy to implement a pattern that closely resembles `Objective-C Categories <http://developer.apple.com/library/ios/#documentation/cocoa/conceptual/ProgrammingWithObjectiveC/CustomizingExistingClasses/CustomizingExistingClasses.html>`_. The idea is to be able to maintain the custom code in a separate file and be able to dynamically merge it with the auto generated code during runtime.

To achieve this for our hypothetical User class, create a file called ``User.js`` in directory ``pdemo/data/extension``, this will provide the namespace ``pdemo.data.extension.User`` and depend on ``pdemo.data.model.User``. 

.. code-block:: javascript

    goog.provide('pdemo.data.extension.User');
    goog.require('pdemo.data.model.User');

    # Closure will ensure that the namespace pdemo.data.extension.User
    # is available here, feel free to extend it

    pdemo.data.model.User.prototype.getFullName = function() { 
        return this.getFirstName() + " " +  this.getLastName();
    };

Now where you want to create an instance of ``pdemo.data.model.User``, use the extension as the dependency ``pdemo.data.model.UserExtension``. This ensures that both the auto generated namespace and your extensions are available.

.. code-block:: javascript

    goog.provide('pdemo.ui.web.Renderer');

    # This will make available the pdemo.data.model.User namespace with your extensions
    goog.require('pdemo.data.extension.User');


Types API
=========

The Types API is a client side implementation of the Prestans types API found on the server side. It assists in directly translating validation rules for Web based clients consuming REST services defined using prestans. Later in this chapter we demonstrate a set of tools that cut out the laborious job of creating client side stubs of your Prestans models.

* ``String``, wraps a string
* ``Integer``, wraps a number
* ``Float``, wraps a number
* ``Boolean``, wraps a boolean
* ``DateTime``, wraps a `goog.date.DateTime <http://closure-library.googlecode.com/svn/docs/class_goog_date_DateTime.html>`_ and includes format configuration from the server side definition.
* ``Array``, extends `goog.iter.Iterator <http://closure-library.googlecode.com/svn/docs/class_goog_iter_Iterator.html>`_ enables you to use ``goog.iter.forEach``, we wrap most of the useful methods provided by Closure iterables.
* ``Model``, wraps JavaScript ``object``
* ``Filter`` is an configurable filter that you can pass with API calls, this translates back into attribute strings, discussed in :doc:`validation`.

.. note:: prestans.types.Integer only support integers in signed 32 bit range as anything outside this range does not work correctly with JavaScript bitwise operators.

Array
-----

``prestans.types.Array`` extends ``goog.iter.Iterator``, allowing you to use the methods from `goog.iter <http://docs.closure-library.googlecode.com/git/namespace_goog_iter.html>`_ including:

* ``goog.iter.filter``
* ``goog.iter.forEach``
* ``goog.iter.limit``

An array takes the following object as its constructor.

.. code-block:: javascript

    {
        elementTemplate: Subclass of prestans.types.Model or instance of prestans.types.Integer, prestans.types.Float, prestans.types.String, prestans.types.Boolean,
        opt_elements: Array of elements to append to the array,
        opt_json: Array of json elements to append to the array,
        opt_minified: Whether or not the json has been minified,
        opt_maxLength: An integer value representing the maximum length of the array,
        opt_minLength: An integer value representing the minimum length of the array
    }

Prestans provides wrappers for the following Google closure `goog.array <http://docs.closure-library.googlecode.com/git/namespace_goog_array.html>`_ methods:

* ``isEmpty`` checks to see if an Array is empty returns a ``Boolean`` 
* ``binarySearch(target, opt_compareFn)`` performs a binary search for an object returns an index ``Number``
* ``binaryInsert(value, opt_compareFn)`` performs a binary insert and returns a ``Boolean``
* ``binaryRemove(value, opt_compareFn)`` performs a binary remove and returns a ``Boolean``
* ``insertAt(obj, opt_i)`` inserts and object at the given index
* ``indexOf(obj, opt_fromIndex)`` returns the index as a ``Number`` for a particular object
* ``removeAt(i)`` removes an object at a particular index and returns a ``Boolean``
* ``removeIf(f, opt_obj)`` removes an object if the supplied function returns ``true`` and returns a ``Boolean``
* ``remove(obj)`` removes the provided object from the collection and returns a ``Boolean``
* ``sort(opt_compareFn)`` performs a sort based on a comparison function
* ``clear`` clears the contents of an array
* ``find(f, opt_obj)`` performs a fund using a user provided function, returns the ``Element`` or ``null``
* ``slice(start, opt_end)`` returns an ``prestans.types.Array`` which is a portion of the original ``Array``
* ``contains(obj)`` returns a ``Boolean`` to see if the collection contains a particular ``object``

Prestans then provides the following additional methods:

* ``getMinLength`` returns a ``Number``
* ``getMaxLength`` returns a ``Number``
* ``append (element)`` returns a ``Boolean``
* ``insertAfter(newValue, existingValue)`` returns a ``Boolean``
* ``length`` returns a ``Number``
* ``containsIf(condition, opt_context)`` returns an ``Element|null``
* ``objectAtIndex(index)`` returns an ``Element``
* ``asArray`` returns an ``Array``
* ``clone`` returns an ``prestans.types.Array``
* ``getJSONObject`` returns an ``Object``
* ``getJSONString`` returns a ``String``


Attribute Change Events
-----------------------

Models generated by Prestans raise the ``prestans.types.Model.EventType.ATTRIBUTE_CHANGED`` event whenever a mutator is fired. You can listen for this event on any instance of a Prestans Model subclass. 

``event.getIdentifier()`` provides you a camel cased representation of the attribute that changed:

.. code-block:: javascript

    var album_ = new pdemo.data.model.Album();

    // Use the Event Handler provided by a Google Closure Component
    this.getHandler().listen(album_, prestans.types.Model.EventType.ATTRIBUTE_CHANGED, function(event){ 

        if(event.getIdentifier() == "name") {
            // name changed, I might update the user interface
        }

    });


Generating Model Code
---------------------

Based on the server model definition Prestans can generate JavaScript versions of your models. You can use this in tandem with the Xhr client to ensure the data you receive from is intact and matches the business rules (hence ensuring nothing went wrong on the way), and that the data you send to the server will pass the validation test.

The generated model code centralizes references to JSON keys allowing you to access the data via accessors and mutators (which also validate the data) allowing `Closure Compiler <http://github.com/google/closure-compiler>`_ to optimise your code.

Prestans provides a utility called pride (Prestans Integrated Development Environment) which resides in `/usr/local/bin` and is responsible for generating the Model stubs. You provide it a reference to the Python file that contains your Model definitions along with target name spaces and paths and for each server Model definition it produces a JavaScript Model or Attribute Filter.

To generate models using pride use the following command:

.. code-block:: bash

    pride gen --template closure.model --model pdemo/rest/models.py --namespace pdemo.data.model --output client/pdemo/data/model

Prestans assumes that your filters files live in the same level as models i.e ``pdemo.data.model`` corresponds to ``pdemo.data.filter``, you can optionally provide a ``--filter-namespace`` to override the default filter namespace

To generate filters using pride use the following command:

.. code-block:: bash

    pride gen --template closure.filter --model pdemo/rest/models.py --namespace pdemo.data.filter --output client/pdemo/data/filter


REST Client
===========

Prestans contains a ready made REST Client to allow you to easily make requests and unpack responses from a Prestans enabled server API. Our client implementation is specific to be used with Google Closure and only speaks `JSON`.

The client has three important parts:

* Request Manager provided by ``prestans.rest.json.Client``, this queues, manages, cancels requests and is responsible for firing callbacks on success and failure. Your application lodges all API call requests with an instance of ``prestans.rest.json.Client``. It's designed to be shared by your entire application.
* Request provided by ``prestans.rest.json.Request`` is a formalised request that can be passed to a Request Manager. The Request constructor accepts a JSON payload with configuration information, this includs partial URL schemes, parameters, optional body and a format for the response. The Request Manager uses the responses format to parse the server response.
* Response provided by ``prestans.rest.json.Response`` encapsulates a server response. It also contains a parsed copy of the server response expressed using Prestans types.

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
        API_CLIENT: new prestans.rest.json.Client({
            baseUrl: "/api",
            opt_numRetries: 0,
            opt_minified: true
        })
    };

.. note:: Minification support is built into our Xhr client. All you have to do is set the optional parameter to ``true`` and the client negotiates with the server.

Composing a Request
-------------------

To place an Xhr request you compose a request by instantiating a ``prestans.rest.Request`` object, it accepts the following parameters as a JSON configuration:

* ``identifier`` unique string identifier for this request type, these are used to cancel requests
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


Example of a GET request which optionally passes two parameters and expects the server to return a set of ``Album`` entities:

.. code-block:: javascript

    var config_ = {
        identifier: "AlbumSearchFetch",
        httpMethod: prestans.net.HttpMethod.GET,
        responseFilter: opt_filter,
        responseModel: pdemo.data.model.Album,
        isArray: true,
        urlFormat: "/album",
        parameters: [
            {
                key: "search_text",
                value: searchText
            },
            {
                key: "limit",
                value: limit
            }
        ]
    };

    var request = prestans.rest.json.Request(config_);

Example of a GET request which fetches a particular entity:

.. code-block:: javascript

    // Optional filter to turn off all attributes bar Album Id and Name
    var opt_filter = new pdemo.data.filter.Album(false);
    opt_filter.enableName();
    opt_filter.enableAlbumId();

    var config_ = {
        identifier: "AlbumFetch",
        httpMethod: prestans.net.HttpMethod.GET,
        responseFilter: opt_filter,
        responseModel: pdemo.data.model.Album,
        isArray: false,
        urlFormat: "/album/%i",
        urlArgs: [albumId]
    };

    var request = prestans.rest.json.Request(config_);

Dispatching a Request
---------------------

Once you have a request object use the ``dispatchRequest`` method on the Request Manager instance to dispatch API calls, it requires the following parameters:

* ``request`` is a ``prestans.rest.json.Request`` object.
* ``callbackSuccessMethod`` which is a reference to a function the Request Manager calls if the API call succeeds, the method will be passed a response object. Ensure you use ``goog.bind`` to bind your function to your namespace.   
* ``callbackFailureMethod`` optional reference to a function the Request Manager calls if the API call fails, this method will be passed a response object with failure information. 
* ``opt_abortPreviousRequests``, optionally asks the Request Manager to cancel all pending requests.

.. code-block:: javascript

    # Assume you have a request object
    pdemo.GLOBALS.API_CLIENT.dispatchRequest(
        request,
        goog.bind(this.successCallback_, this),
        goog.bind(this.failureCallback_, this),
        false
    );

.. note:: Request objects tell the manager if they are willing to be aborted, this is configurable per request lodged with the manager.

The second method the Request Manager provides is ``abortAllPendingRequests``, this accepts no parameters and is responsible for aborting any currently queued connections. The failure callback is not fired when requests are aborted.

Working with Responses
----------------------

The Xhr calls back the nominated functions on success and failure. Prestans passes an instance of ``prestans.rest.json.Response`` which has access to th following:

* ``requestIdentifier`` The string identifier for the request type,
* ``statusCode`` HTTP status code,
* ``responseModel`` Class used to unpack response body,
* ``arrayElementTemplate`` prestans.types.Model subclass
* ``responseModelElementTemplates``
* ``responseBody`` JSON Object (Optional), this won't be available for failures

In case of a successful request you will be able to retrieve the parsed Prestans JavaScript model using the ``getUnpackedBody()`` method.

.. code-block:: javascript

    /**
     * @private
     */
    pdemo.ui.album.Renderer.prototype.successCallback_ = function(response) {

        // response.getUnpackedBody() will return a parsed Prestans model
        // based on the rules you defined in the request
        console.log(response.getUnpackedBody());

    };

In case of failures you get access to the response which has the status code and the original body (if any) of the response. For failures you are expected to do what your application deems appropriate.

Xhr Communication Events
------------------------

The Request Manager raises the following events. These come in handy if your application requires global UI interactions e.g a Modal popup if network communication fails, or notification messages on success.

* ``prestans.rest.json.Client.EventType.RESPONSE``, raised when a round trip succeeds, this would be raised even if your API raised an error code, e.g Bad Request or Service Unavailable.
* ``prestans.rest.json.Client.EventType.FAILURE`` raised if a round trip fails.

Example of using ``goog.events.EventHandler`` to listen to the Failure event:

.. code-block:: javascript

    goog.require('goog.events.EventHandler');

    # and somewhere in one of your functions
    this.eventHandler = new goog.events.EventHandler(this);
    this.eventHandler_.listen(pdemo.GLOBALS.API_CLIENT, prestans.rest.json.Client.EventType.FAILURE, this.handleFailure_);

The ``event`` object passed to the end points is of type ``prestans.rest.json.Client.Event`` a subclass of ``goog.events.Event``. Call ``getResponse`` method on the event to get the ``Response`` object, this will give you access all the information about the request and it's outcome.
