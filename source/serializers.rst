===========
Serializers
===========



Writing your own serializer
===========================

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


And hence the serizliaer would look like

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