===========================
Routing & Handling Requests
===========================

Web Server Gateway Interface or WSGI (:pep:`333`) is the glue between a Web Server and your Python Web application. The responding application simply has to be a Python `callable <http://docs.python.org/2/library/functions.html#callable>`_ (a python function or a class that implements the ``__call__`` method). Each *callable* is passed the Web server environment (much like CGI applications) and a ``start_response``. 

If you are not familiar with WSGI we recommend reading `Armin Ronarcher <http://lucumr.pocoo.org/>`_'s `introduction to WSGI <http://lucumr.pocoo.org/2007/5/21/getting-started-with-wsgi/>`_. We also have a great collection of :doc:`reference_material` on Python Web development.

WSGI interfaces will generally handover requests that match a URL pattern to the mapped WSGI callable. From the callable is responsible for dispatching the request to the appropriate handler based on part of the URL, HTTP verb, headers or any other property of an HTTP request or a combination properties. This middle ware code is refered to as a request router and Prestans provides one of it's own.

prestans makes use of standard HTTP headers for content negotiation. In addition it uses a handful of custom headers that the client can use to control the Prestans based API's behavior (features include Content Minification and Attribute Subsets for requests and responses). We'll first introduce you to the relevant HTTP and how it effects your API requests followed by how you can handle API requests in prestans.

Serializers & DeSerializers
===========================

Serializers and DeSerializers are pluggable Prestans constructs that assist the framework in packing or unpacking data. Serialzier or deserializer handle content of a particular mime type and are generally wrappers to vocabularies already available in Python (although it possible to write a custom serializer entirely in Python). Serializers always write out a parseable version of models.

prestans application may speak as many vocabularies as they wish; vocabularies can also be local to handlers (as opposed to applicaiton wide). You must also define a default format.

Each request must send an ``Accept`` header for Prestans to decide the response format. If the registered handler cannot respond in the requeted format Prestans raises an ``UnsupportedVocabularyError`` exception inturn producing a ``501 Not Implemented`` response. All Prestans APIs have a set of default formats all handlers accept, each end-point might accept additional formats.

If a request has send a body (e.g ``PUT``, ``POST``) you must send a ``Content-Type`` header to declare the format in use. If you do not send a ``Content-Type`` header Prestans will attempt to use the default deserializer to deserialize the body. If the ``Content-Type`` is not supported by the API an ``UnsupportedContentTypeError`` exception is raised inturn producing a ``501 Not Implemented`` response.

Routing Requests
================

prestans is built right on top of WSGI and ships with it's own WSGI Request Router. The router is responsible for parsing the HTTP request, setting up a HTTP response, setup a logger (if one wasn't provided as part of the configuraiton), finally check to see if the API services the requested URL and hand the request over to the handler.

The handler is responsible for constructing the response and one return the router, asks the response to serialize itself. If an exception is raised (see :doc:`framework_design`) is raised by the framework (because it couldn't parse the request or response) or the handler, the router where appropriate, writes a detailed error trace back to the client.

prestans verbosely logs events per API request. Your system logger is responsible for verbosity of the logger.

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

The router is provided by the ``prestans.rest`` package. It's the keeper of all Prestans API requests, it (with the help of other members of the ``prestans.rest`` package) parases the HTTP request, setups the appropriate handler and hands over control.

The router also handles error states raised by the API and informs the requesting client to what went wrong. These can include issues with parsing the request or exceptions raised by the handler (this is covered later in the chapter) while dealing with the request.

The constructor takes the following parameters:

* ``routes`` a list of tuples that maps a URL with 
* ``serializers`` takes a list of serializer instances, if you omit this parameter Prestans will assign JSON as serializer to the API.
* ``default_serializer`` takes a serializer instance which it uses if the client does not provide an ``Accept`` header.
* ``deserializers``, a set of deserializers that the clients use via the ``Content-Type`` header, this default to ``None`` and will result in Prestans using JSON as the default deserializer. 
* ``default_deserializer``, default serializer to be used if a client doesn't provide a ``Content-Type`` header, defaults to ``None`` which results in Prestans using the JSON deserializer. 
* ``charset``, to be used to parse strings, defaulted to ``utf-8``
* ``application_name`` the name of your API, ``prestans``
* ``logger``, an instance of a Python logger, defaulted to ``None`` which results in Prestans creating a default logger instance.
* ``debug``, runs Prestans under debug mode (results in increased logging, error reporting), it's defaulted to ``False``

.. code-block:: python

    import prestans.rest
    import myapp.rest.handlers

    api = prestans.rest.RequestRouter(routes=[
            (r'/([0-9]+)', myapp.rest.handlers.DefaultHandler)
        ], 
        serializers=[prestans.serializer.JSON()],
        default_serializer=prestans.serializer.JSON(),
        deserializers=[prestans.deserializer.JSON()],
        default_deserializer=prestans.deserializer.JSON(),
        charset="utf-8",
        application_name="music-db", 
        logger=None,
        debug=True)


The router is a standalone WSGI application you pass onto server environment. 

If your application prefers a diaclet other than JSON as it's default, ensure you configure this as part of the router. It's recommended that the default diaclet for serialization and deserialization is the same.

The default logger uses is a configured `Python Logger <http://docs.python.org/howto/logging>`_, it logs in detail the lifecycle of a request along with the requested URL. Refer to documentation on how to configure your system logger to control verbosity.

Once your router is setup, Prestans is ready to route requests to nominated handlers.

If were deploying under mod_wsgi, your the above would be the contents of your WSGI file. mod_wsgi requires the endpoint to be called ``application``. The WSGI configuration variable might look something like.

.. code-block:: apache

    WSGIScriptAliasMatch    ^/api/(.*)  /srv/musicdb/api.wsgi

Under AppEngine, if this above was declared under a script named ``entry.py``, you would reference it in ``app.yaml`` as follows:

.. code-block:: yaml

    - url: /api/.*
      script: entry.api
      login: required

Handling Requests
=================

REST requests primarily use the following HTTP verbs to handle requests:

* ``HEAD`` to check if the entity the client has is still current
* ``GET`` to retrieve entities
* ``POST`` to create a new entity
* ``PUT`` to update an entity
* ``PATCH`` to update part of an entity
* ``DELETE`` to delete an entity

prestans maps each one of these verbs to a python function of the same name in your REST handler class. Each REST request handler in your application derives from ``prestans.rest.RequestHandler``. Unless your handler overrides the functions ``get``, ``post``, ``put``, ``patch``, ``delete`` the base implementation tells the Prestans router that the requested end point does not support the particular HTTP verb.

Your handler must accept an equal number of parameters as defined the router regular expression.

Our Regex premier highlights the use of two handlers per entity, one deals with collections the other entities. APIs generally let clients get a collection of entities, add to a collection and get a particular entity, update an entity or delete an entity. The later require an identifier for the entity, where as the collection does not.

* ``/api/album/([0-9]+)/track/*``
* ``/api/album/([0-9]+)/track/([0-9]+)``

This is no way says that your API can't provide an endpoint to delete all entities of a type or update a collection of entities, in which instances your collection handler would implement the appropriate HTTP verb handlers.

A typical collection handlers would typically look like (implementing ``GET`` and ``POST`` and does not require an identifier):

.. code-block:: python

    import prestans.rest
    import prestans.parser

    import myapp.rest.models

    class MyCollectionRESTRequestHandler(prestans.rest.RequestHandler):

        __parser_config__ = prestans.parser.Config(
            GET=prestans.parser.VerbConfig(
                response_template=prestans.types.Array(element_template=myapp.rest.models.Track())
            ),
            POST=prestans.parser.VerbConfig(
                body_template=myapp.rest.models.Track(),
                response_template=myapp.rest.models.Track()
            )
        )

        def get(self):
            ... return a collection of entities

        def post(sef):
            ... add a new type


A typical entity handler would look like (implementing ``GET``, ``PUT`` and ``DELETE`` expecting an identifier):

.. code-block:: python

    import prestans.rest
    import prestans.parser

    import myapp.rest.models

    class MyEntityRESTRequestHandler(prestans.rest.RequestHandler):

        __parser_config__ = prestans.parser.Config(
            GET=prestans.parser.VerbConfig(
                response_template=myapp.rest.models.Track()
            ),
            PUT=prestans.parser.VerbConfig(
                body_template=myapp.rest.models.Track(),
            )
        )

        def get(self, track_id):
            ... return an individual entity

        def put(self, track_id):
            ... update an entity

        def delete(self, track_id):
            ... delete an entity


Notice that since deleting an entity only requires an identifier, and does not have to parse the body of a request. The update request can also choose to use attribute filters to pass in partial objects.

.. note:: At this point if you'd rather learn about how to parse requests and responses, then head to the chapter on :doc:`validation`. This chapter continues to talk about how handlers work assuming you are going to read the chapter on validation shortly after.

Each handler allows accessing the environment as follows:

* ``self.request`` is the parsed request based on the rules defined by your handler, this is an instance of ``prestans.rest.Request``
* ``self.response`` is response Prestans will eventually write out to the client, this is an instance of ``prestans.rest.Response``
* ``self.logger`` is an instance of the logger the API expects you to write any information to, this must be an instance of a Python logger
* ``self.debug`` is a boolean value passed on by the router to indicate if we are running in debug mode

Each request handler instance is run in the following order (all of these methods can be overridden by your handler):

* ``register_seriailzers`` is called on the handler, this allows the handler to a list of additional serializers it would like to use
* ``register_deserializers`` is called on the handler, this allows the handler to a list of additional deserializers it would like to use
* ``handler_will_run`` is called, perform any handler specific warm up acts here
* The function that corresponds to the requests HTTP verb is called
* ``handler_did_run`` is called, perform any handler specific tear downs here

``prestans.rest.RequestHandler`` can be subclassed as your project's Base Handler class, this generally contains common code e.g getting access to a database sessions, etc. A SQLAlchemy specific example would look like:

.. code-block:: python

    import prestans.rest
    import prestans.parser

    import myapp.rest.models

    class Base(prestans.rest.RequestHandler):

        def handler_will_run(self):
            self.db_session = myapp.db.Session()
            self.__provider_config__.authentication = myapp.rest.auth.AuthContextProvider(self.request.environ, self.db_session)

        def handler_did_run(self):
           myapp.db.Session.remove()

        @property
        def user_profile(self):
            return self.__provider_config__.authentication.get_current_user()

        @property
        def auth_context(self):
            return self.__provider_config__.authentication

.. note:: You'd typically place this in ``myapp.rest.handlers.__init__.py`` and place all your handlers grouped by entity type in that package.

Constructing Response
=====================

The end result of all handler call is to send a response back to the client. This can be as simple as a status code, or as elaborate as a group of entities. Prestans is unforgiving (unless requested otherwise) while accepting requests and writing responses.

In accordance with the REST standard, each handler must declare what sort of entities (if any) the handler will return if it successfully processes a request. We will cover error scenarios later in this chapter.

Declaration of response types are defined as part of your parser configuration per HTTP verb. Handlers typically return a collection of entities, defined as:

.. code-block:: python

    import prestans.rest
    import prestans.parser
    import prestans.types

    import myapp.rest.models

    class MyEntityRESTRequestHandler(prestans.rest.RequestHandler):

        __parser_config__ = prestans.parser.Config(
            GET=prestans.parser.VerbConfig(
                response_template=prestans.types.Array(element_template=myapp.rest.models.Album())
            )
        )

        def get(self):

            albums = prestans.types.Array(element_template=myapp.rest.models.Album())
            albums.append(myacc.rest.models.Album(name="Journeyman", artist="Eric Clapton"))
            albums.append(myacc.rest.models.Album(name="Dark Side of the Moon", artist="Pink Floyd"))
            return albums            

or an individual entity, defined as:

.. code-block:: python

    import prestans.rest
    import prestans.parser

    import myapp.rest.models

    class MyEntityRESTRequestHandler(prestans.rest.RequestHandler):

        __parser_config__ = prestans.parser.Config(
            GET=prestans.parser.VerbConfig(
                response_template=myapp.rest.models.Album()
            )
        )

        def get(self, album_id):
            
            return myapp.rest.models.Album(name="Journeyman", artist="Eric Clapton")

More often than not, the content your handler sends back, would have been read queried from a persistent data store. Sending persistent data verbatim nearly never fits the user case. A useful API sends back appropriate amount of information to the client to make the request useful without bloating the response. This becomes a cases by case consideration of what a request handler sends back. Sending out persistent objects verbatim could sometimes pose to be a security threat.

prestans requires you to transform each persistent object into a REST model. To ease this tedious task Prestans provides a feature called :doc:`data_adapters`. Data Adapters perform the simple task of converting persistent instances or collections of persistent instances to paired Prestans REST models, while ensuring that the persistent data matches the validation rules defined by your API.

Data Adapters are specific to backends, and it's possible to write your own your backend specific data adapter. All of this is discussed in the chapter dedicated to :doc:`data_adapters`.

Minifying Content
-----------------

prestans tries to make your API as efficient as possible. Minimizing content size is one of these tricks. Complimentary to Attribute Filters (which allows clients to dynamically turn attributes on or off in the response) is response key minification.

This is particularly useful for large amounts of repetitive data, e.g several hundred order lines.

Setting the ``Prestans-Minification`` header to ``On`` is all that's required to use this feature. This is a per request setting and is set to ``Off`` by default.

prestans also sends ``Prestans-Minification-Map`` header back containing a one to one map of the original attribute names it's minified counterpart.

.. note:: Our Google Closure extensions provides a JSON REST Client, which can automatically unpack minified requests to fully formed Prestans client side models.

Serving Binary Content
----------------------

It's perfectly legitimate for your REST handler to return binary content. Prestans provides a built in model to assign to your handler's ``VerbConfig``. 

Your handler must return an instance of ``prestans.types.BinaryContent``, and Prestans will do what's right to deliver binary content.

Instances of ``BinaryContent`` accept the following parameters:

* ``mime_type``, the mime type of the file that you are sending back
* ``file_name``, the filename that Prestans is to send back in the HTTP header, this is what the browser thinks the name of the file is. File names can be generated by applications or they might have been stored as meta information when the file was uploaded by the user
* ``as_attachment``, tells Prestans if the file is to be delivered as an attachment (forces the user to save the file) or deliver it inline (generally opens in the browser).
* ``contents``, binary contents that you've read up from disk or generated.

.. code-block:: python

    import prestans.types

    class Download(st.cs.rest.handlers.Base):

        __parser_config__ = prestans.parser.Config(
            GET=prestans.parser.VerbConfig(
                response_template=prestans.types.BinaryResponse()
            )
        )

        def get(self, document_id):

            file_contents = open(file_path).read()

            self.response.status = prestans.http.STATUS.OK
            self.response.body = prestans.types.BinaryResponse(
                mime_type='application/pdf', 
                file_name='invoice.pdf', 
                as_attachment=True,
                contents=file_contents)

If you set ``as_attachment`` to False the file will be delivered inline into the browser. It's up to the browser to handle the content properly.

Raising Exceptions
==================

As alluded to in our :doc:`api_design` chapter, Prestans provides two distinct set of Exceptions. The first raised if you've configured your API incorrectly and the later used to send back meaningful error messages to the requesting client.

This section deals with Exceptions that Prestans expects you to use to raise meaningful REST error messages. These are generally caused by the client sending you an inappropriate e.g the logged in user is not allowed to access or update an entity, or something simply not being found on the persistent data store.

.. note:: :pep:`008` recommends that Exceptions that are errors should end with the Error suffix.

You do not have to raise exceptions for request and response data validation. If the data does not match the rules defined by your models, parameter sets or attribute filters, it's prestans' responsibility to graceful fail and respond back to the client.

Unsupported Vocabulary
----------------------



Data Validation Exceptions
--------------------------

* ``RequiredAttributeError``
* ``ParseFailedError``
* ``LessThanMinimumError``
* ``MoreThanMaximumError``
* ``InvalidChoiceError``
* ``UnacceptableLengthError``
* ``InvalidType``
* ``MissingParameterError``
* ``InvalidFormatError``
* ``InvalidMetaValueError``
* ``UnregisteredAdapterError``

Parser Exceptions
-----------------

* ``UnimplementedVerbError``
* ``NoEndpointError``
* ``AuthenticationError``
* ``AuthorizationError``
* ``SerializationFailedError``
* ``DeSerializationFailedError``
* ``AttributeFilterDiffers``
* ``InconsistentPersistentDataError``


Handler Exceptions
------------------

The following exceptions are designed

* ``ServiceUnavailable``
* ``BadRequest``
* ``Conflict``
* ``NotFound``
* ``Unauthorized``
* ``MovedPermanently``
* ``PaymentRequired``
* ``Forbidden``

