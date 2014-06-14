===============================
Validating Requests & Responses
===============================

A robust and secure API is responsible for validating all incoming and outgoing data to ensure that it matches the business rules. Validation is at the heart of prestans design, it's a cornerstone feature. 

APIs have three major areas where data validation is paramount:

* Query String parameters supplied to the API as key value pairs in the URL ``http://localhost/api/album?offset=10&limit=20``
* Serialized data in the body of the request, which is unpacked and made available for use to the handler
* Data that's serialized down to the client as the response to each request. This is typically but not necessarily read back from persistent data stores.

prestans allows each request handler to elegantly define the rules it wants to adhere to by declaring what we refer to as a ``VerbConfig`` (the Verb refers to the HTTP verb). 

Setting up validation rules
---------------------------


.. code-block:: python

    import prestans.rest
    import prestans.parser

    import musicdb.rest.models


    class CollectionRequestHandler(prestans.rest.RequestHandler):

        __verb_config__ = prestans.parser.Config(
            POST =  prestans.parser.VerbConfig(
                body_template=musicdb.rest.models.Album()
            )            
        )

        def post(self):
            ... do stuff here

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


Parameters
^^^^^^^^^^

Request Body
^^^^^^^^^^^^

Response Body
^^^^^^^^^^^^^




Attribute Filters
-----------------


