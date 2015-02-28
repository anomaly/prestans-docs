===============================
Validating Requests & Responses
===============================

A robust and secure API is responsible for validating all incoming and outgoing data to ensure that it matches business rules. Validation is at the heart of prestans design and a cornerstone feature. 

APIs have three major areas where data validation is paramount:

* Query String parameters supplied to the API as key value pairs in the URL ``http://localhost/api/album?offset=10&limit=20``
* Serialized data in the body of the request, which is unpacked and made available for use to the handler
* Data that's serialized down to the client as the response to each request. This is typically but not necessarily read back from persistent data stores.

prestans allows each request handler to elegantly define the rules it wants to adhere to by declaring what we refer to as a ``VerbConfig`` (the Verb refers to the HTTP verb). 

.. _attribute_filters:

Attribute Filters
-----------------

Attribute Filters are prestans way of making temporary exceptions to validation rules otherwise defined by prestans ``Models``. Quality of code written using prestans thrives on strong validation. Certainly uses cases in every application demands relaxing rules temporarily. Attribute filters are used both incoming and outgoing data.

At it's heart Attribute Filters are a configuration template for exceptions prestans is to make while parsing data. Attribute Filters contain a boolean flag for each attribute defined in instances that will be parsed by prestans. They can be created by subclassing ``prestans.parser.AttributeFilter`` and it containing an identical structure to the corresponding model with boolean flags to denote visibility during a parse operation. 

However we recommend using our convenience method that dynamically creates a filter based on a model (unless of course you have non trivial use case):

.. code-block:: python
    
    attribute_fitler = prestans.parser.AttributeFilter.from_model(model_instance=mysicdb.rest.models.Album(), default_value=False)

the above will generate an attribute filter with all attributes turned off for parsing. We can then selectively turn attributes on by setting the keys to ``True``:

.. code-block:: python

    attribute_filter.id = True
    attribute_filter.name = True

The attribute filter will have a corresponding definition for every attribute visible in the ``Model`` with the default boolean value assigned to it. This goes for children objects. 

If a child attribute is a collection then the child Attribute Filter is based on an instance that would appear in the collection, this allows fine grained control of toggling parse rules. Assigning a ``boolean`` value to a collection applies the state to all attributes of it's instances.

Setting up validation rules
---------------------------

Validation rules are set up per HTTP verb your handler intends to service. By default there are no validation rules defined for any HTTP verb, this does not mean that your handler can't respond to a particular verb, it simply means that prestans takes no responsibility of validating incoming or outgoing data. By design if you wish to send data back to the client prestans insist on validating what a handler sends down, however it's perfectly valid for a handler to return no content (which is what prestans expects if you aren't specific).

Each handler has a meta attribute called __verb_config__ this must be an instance of ``prestans.parser.Config`` which accepts six named parameters one for each supported HTTP verb (``HEAD``, ``GET``, ``POST``, ``PUT``, ``DELETE``, ``PATCH``) each one of which must be an instance of ``prestans.parser.VerbConfig``. A ``VerbConfig`` accepts the following named parameters (not all of them are supported across all HTTP verbs):

* ``response_template`` an instance of a ``prestans.types.DataCollection`` subclass i.e a Model or an Array of prestans ``DataType``. This is what prestans will use to validate the response your handler sends back to the client.
* ``response_attribute_filter_default_value`` prestans automatically creates an attribute filter based on the ``response_template`` by default prestans exposes all it's attributes in the response, setting this to ``False`` will hide all attributes be default. Your handler code is responsible for toggling visibility in either instance.
* ``parameter_sets`` an array of ``prestans.parser.ParameterSet`` instances (see :ref:`parameter_sets`)
* ``body_template`` an instance of a ``prestans.types.DataCollection`` subclass i.e a Model or an Array of prestans ``DataType``, this is what prestans will use to validate the request sent to your handler. If validation of the incoming data fails, prestans will not execute the associated verb in your handler.
* ``request_attribute_filter`` is an attribute filter used to relax or tighten rules for the incoming data. This is particularly useful if you want to use portions of a model. Particularly useful for ``UPDATE`` requests.

.. code-block:: python

    import prestans.rest
    import prestans.parser

    import musicdb.rest.models


    class CollectionRequestHandler(prestans.rest.RequestHandler):

        __verb_config__ = prestans.parser.Config(
            GET =  prestans.parser.VerbConfig(
                body_template=prestans.types.Array(element_template=musicdb.rest.models.Album())
            ),            
            POST =  prestans.parser.VerbConfig(
                body_template=musicdb.rest.models.Album()
            )            
        )

        def get(self):
            ... do stuff here

        def post(self):
            ... do stuff here

prestans is aggressive when it comes to validating requests and responses. However in the cases where you wish to relax the rules we recommend that you use :ref:`attribute_filters`. You can define an ``AttributeFilter`` in context and assign it to the appropriate ``VerbConfig``.

.. code-block:: python

    update_filter = prestans.parser.AttributeFitler(model_instance=musicdb.prest.models.Album(), default_value=False)
    update_filter.name = True

    class EntityRequestHandler(prestans.rest.RequestHandler):

        __verb_config__ = prestans.parser.Config( 
            GET = prestans.parser.VerbConfig(
                response_template=musicdb.rest.models.Album(),
                response_attribute_filter_default_value=False,
                parameters_sets=[]
            ),
            PUT =  prestans.parser.VerbConfig(
                body_template=musicdb.rest.models.Album(),
                request_attribute_fitler=update_filter
            )
        )

        def get(self, album_id):
            ... do stuff here

        def put(self, album_id):
            ... do stuff here

        def delete(self, album_id):
            ... do stuff here

Lastly a reminder parameters that were part of your URL scheme will be passed in as positional arguments to your handler verb (see :doc:`handlers`). prestans runs your handler code if the the request succeeds to parse and will only respond back to the client if the response you intend to return passes the validation test.

Working with parsed data
------------------------

The following sections detail how you access the parsed data and how you provide prestans with a valid response to send back to the client. Remember that your handler's objective is to send back information the client can reliably use.

.. _parameter_sets:

Parameters Sets
^^^^^^^^^^^^^^^

``ParmeterSets`` refer to sets of data sent as key value pairs in the query string. Typically if you handler is expecting data as part of the query string you would expect it to be follow similar patterns as ``Models``. prestans extends the use of it's ``types`` (see :doc:`types`) to validate data passed in a query string.

Each ``ParameterSet`` is made of a group of keys that you're expecting along with rules to be used to parse the value. ``ParameterSets`` are defined by subclassing ``prestans.parser.ParameterSet``.

.. code-block:: python

    class SearchByKeywordParameterSet(prestans.parser.ParameterSet):

        keyword = prestans.types.String(min_length=5)
        offset = prestans.types.Integer(defauflt=0)
        limit = prestans.types.Integer(default=10)

    class SearchByCategoryParameterSet(prestans.parser.ParameterSet):

        category_id = prestans.types.Integer(min_length=5)
        offset = prestans.types.Integer(defauflt=0)
        limit = prestans.types.Integer(default=10)

these would then be assigned to your handler's ``VerbConfig`` as follows:

.. code-block:: python

    __verb_config__ = prestans.parser.Config( 
        GET = prestans.parser.VerbConfig(
            response_template=musicdb.rest.models.Album(),
            response_attribute_filter_default_value=False,
            parameters_sets=[SearchByKeywordParameterSet(), SearchByCategoryParameterSet()]
        ),
        PUT =  prestans.parser.VerbConfig(
            body_template=musicdb.rest.models.Album(),
            request_attribute_fitler=update_filter
        )
    )

.. note:: Parameter Set can only use basic data types i.e ``Strings``, ``Integer``, ``Float``, ``Date``, ``Time``, ``DateTime``. 

Using serialized data as values for query string keys is not a good idea. 

All web servers have limitations on how large query strings can be, if you experience issues with sending information via the query string you should check your web server configuration before attempting to debug your code.

For each request:

* If the data provided as part of a query string matches, prestans will make an instance of that ``ParameterSet`` available at ``self.request.parameter_set``. 
* If a query string would result in matching more than one ``ParameterSet`` prestans will stop parsing at the first match and make it available to your handler
* Failure in matching a ``ParameterSet`` still results in your handler code being called. prestans would simply set ``self.request.parameter_set`` to ``None``.

You can access the attributes defined in your ``ParameterSet`` as you would any ordinary Python object.

If your handler assigned multiple ``ParameterSets`` to a handler ``VerbConfig`` you can always check for the ``type`` of ``self.request.paramter_set`` for conditional code execution.

Request Body
^^^^^^^^^^^^

By assigning a ``DataCollection`` object to the ``body_template`` configuration of a ``VerbConfig`` asks prestans to strictly parse the data received as part of every request. If the data sent as part of the body successfully parses your handler code is executed and the parsed object is available as ``self.request.parsed_body``.

Should you wish to relax the rules of a model for particular use cases you should consider using :ref:`attribute_filters` as opposed to relaxing the validation rules. The attribute filter used while parsing the request is available as ``self.request.attribute_filter``.

.. note:: ``GET`` requests cannot have a request body and prestans will not attempt to parse the body for ``GET`` requests.

Your handler code can access the ``parsed_body`` as a regular Python object. If your handler accepts collections of elements then prestans makes available a prestans Array in the ``parsed_body`` which is a Python iterable.

Response Body
^^^^^^^^^^^^^

Once your handler has completed what it needed to do, it can optionally return a response body. If you aren't returning a body then your handler is simply required to set ``self.response.status`` to a valid HTTP status, prestans has wrapper constants available in ``prestans.http.STATUS``.

Prestans will respect the ``response_template`` configuration set by your handler's ``VerbConfig``. You must return an object that matches the rules. There are generally two scenarios:

* Your handler will return an entity that is an instance of a ``prestans.rest.Model`` subclass. This is typically the case for Entity handlers.
* Your handler returns a collection (i.e a prestans Array) which contains instances of a prestans ``DataType`` usually a Model. This is typically the case for Collection handlers.

REST end-points must always return the same ``type`` of response. This is discussed in detail in :doc:api_design.

.. note :: prestans does not allow sending down instances of python types because they do not conform to a serialization format making it difficult for the client to determine the reliability of the response.

.. code-block:: python

        def get(self, album_id):

            ... assuming you have an object that you can return

            self.response.status = prestans.http.STATUS.OK
            self.response.body = new musicdb.rest.models.Album(name="Journeyman", artist="Eric Clapton")

If you read your response from a persistent store you would be required to convert that object (typically by copying the values) to a similarly formatted prestans REST model. prestans features :doc:`data_adapters` which automate this process.

prestans allows clients to dynamically configure the response payload by sending a serialized version of an Attribute Filter as the HTTP header ``Prestans-Response-Attribute-List``. This allows the client to adjust the response from your API without you having to do extra work. Of course the server has the final say, if your handler outright sets a rule there's nothing a client can do to override it.

The response object that your handler has access to has a reference to an :ref:`attribute_filters` which is made up of the rules defined by your ``response_template`` with the client's request preferences applied, accessible at ``self.response.attribute_filter``. If your handler changes the state of the attribute filter before the verb method returns, prestans used the modified state of the attribute filter, giving you the final say.



