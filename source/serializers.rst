===========
Serializers
===========

REST applications should be able to respond to clients in more than one format. Sound a theory but practically REST applications speak one major format and make exceptions to the rule, e.g an exportable report for download in CSV, and delivering the same data to a client in JSON for visualization. Serializers may also require the REST application to format their data in very different ways, e.g JSON would be a tree style response where as CSV would be linear.

Many frameworks expect the REST handler to choose how each response should be serialized (based on URL patters), we end up creating more work (not to mention large if else blocks) for the typical scenarios to accomodate the exceptions. pretans takes a slightly different appraoch to this problem and pairs serializers ``RESTApplication`` implementations. prestans REST handlers return either a prestans or Python type as their response and rely on the serializer to do their work, this enables you to reuse handlers with multiple serializers simply by referring to the same REST handler class from multiple end points.

.. note:: The gotcha is mixing and matching serializers to REST handlers that follow similar structures. E.g Tree structures opposed to linear structures.

To make exceptions to the rule it's recommended you create a separate URL scheme that fits the serialization format, mapped to appropriate and often exclusive REST handlers. Not all your business logic work should be done in your REST handlers, if they are reusable consider pushing the code into a common class or if you are using ORM layers consider distributing it based on the persistent models you are working with.

Serializers follow the prestans Provider (see :doc:`concepts`) paragradim. A serializer provides a wrapper on a pair of functions to write and read a data exchange format. It's recommended you use standard Python functions to perform the serialize and unserialize operations for performance. However if you need to you can write a pure Python implementation of your serializer.

Serializers are never used directly, it's always paired with a RESTApplication. We current provide support for the following serialization formats:

* JSON via ``prestans.serializers.JSONSerializer``, paired with ``prestans.rest.JSONRESTApplication``
* YAML via ``prestans.serializers.YAMLSerializer``, paired with ``prestans.rest.YAMLRESTApplication``

Writing your own serializer
---------------------------

The Provider paradigm provides a really simple way for you to write your own serializers and pair them with custom ``RESTApplication``. This section discusses and demonstrates how simple it is to write your own serializers. 

Serializers extend from ``prestans.serializers.Serializer`` and expect you to implement these three methods (our example discusses the JSON serializer we ship with prestans):

* ``loads`` is responsible for unserializing input data for your application, it's provided a Python ``string`` and is expected to return an python data type, usually an iterable.

* ``dumps`` provided a pure Python object that may be itterable and needs to be serialized. This function must return the serialized data back to the caller. prestans is responsible for writing the data back to the client

* ``get_content_type`` is expected to send back the mime type of the serialization format in use as a python ``string``

prestans provides  ``prestans.serializers.UnserializationFailedException`` and gracefully handles the client response, if the serialization process has problems it's recommended you raise this exception, with a meaningful message.

.. code-block:: python

    ## @brief Provider for JSON based serializer
    #
    class JSONSerializer(Serializer):
        
        ## @brief loads method for JSON serializer
        #
        # @param self The object pointer
        # @param input_string
        #
        @classmethod
        def loads(self, input_string):
            import json
            parsed_json = None
            try:
                parsed_json = json.loads(input_string)
            except Exception, exp:
                raise UnserializationFailedException('Input Body data is not valid JSON')
                
            return parsed_json

        ## @brief dumps method for JSON serializer
        #
        # @param self The object pointer
        # @param serializable_object
        #
        @classmethod
        def dumps(self, serializable_object):
            import json
            return json.dumps(serializable_object)
            
        @classmethod
        def get_content_type(self):
            return 'application/json'

You can instantiate this class and test it works on the Python interactive interface. Once you are confident that your serializer wrapper works, proceed to pairing it with a RESTApplication that you create.

Pairing it with your REST Application
-------------------------------------

Nearly all of the ``RESTApplication`` logic and the prestans API lifecycle is encapsulate in ``prestans.rest.RESTApplication`` (this class is not paired with a serializer and never meant to be used directly). All custom implementations extend from ``prestans.rest.RESTApplication`` and expect them to construct a ``Request`` and ``Response`` to be used by the API lifecycle. These objects expect the serializer ``class`` they are meant to use.

REST Application implementations are required to override the following class methods:

* ``make_request`` expected to return an instance of ``prestans.rest.Request``, and is passed in a reference to the WSGI environ
* ``make_response`` expected to return an instance of ``prestans.rest.Response``

The following example is of the commonly used ``JSONRESTApplication`` taken from the ``prestans.rest`` package:

.. code-block:: python

    ## @brief REST Application Gateway that speaks JSON
    #
    class JSONRESTApplication(RESTApplication):

        @classmethod
        def make_request(self, environ):
            rest_request = Request(environ, 
                                   serializer=prestans.serializers.JSONSerializer)
            return rest_request

        @classmethod
        def make_response(self):
            rest_response = Response(serializer=prestans.serializers.JSONSerializer)
            return rest_response

.. note:: While it's possible, it's considered to be against the prestans design principles to pair a serializer with multiple RESTApplication implementations. 