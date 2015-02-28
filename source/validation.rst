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



