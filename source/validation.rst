====================
Validating  Requests
====================

prestans provides a well defined way of parsing out set of Parameters in the URL, and or ensure that the request body is properly formed and conforms to the rules defined by your application. Validation is one of the biggest time savers provided by prestans, you can reliably assume that if your handler method is being called, the data available to it is valid and conforms to the rules defined by you.

Each handler can be assigned an instance of a ``RequestParser`` subclass. Each HTTP method has a corresponding variable that expects an instance of ``ParserRuleSet``.

Each ``ParserRuleSet`` can take one of four parameters, all parameters are optional:

* ``parameter_sets``, this takes an Array of ``ParameterSet`` objects
* ``body_template``, accepts a subclass of ``DataCollection`` (most commonly used are ``Model`` subclass or a prestans``Array``), this is used to validate the raw data sent in the body of an HTTP request. GET requests do not have a request body.
* ``response_attribute_filter_template``, accepts a subclass of ``DataCollection``, this assists in clients asking prestans to omit attributes in it's response.
* ``request_attribute_filter``, accepts a subclass of ``DataCollection``, this allows you to relax the rules for a Model for a particular handler and method. This is dicussed later in this section and proves extremely handy for PUT, PATCH requests.

.. code-block:: python

    class MyRequestParser(prestans.parsers.RequestParser):

        GET = prestans.parsers.ParserRuleSet(
            parameter_sets = [
                KeywordSearchParameterSet()
            ]
        )

        POST = prestans.parsers.ParserRuleSet(
            body_template=project.rest.models.MyModel(),
        )

        PUT = prestans.parsers.ParserRuleSet(
            body_template=project.rest.models.MyModel(),
        )

The above parser description, if assigned to a request handler would ensure that POST and PUT requests have a model in the body that matches the rules defined by ``MyModel``. By default prestans is **unforgiving** in matching the body of a request, if the validation fails, prestans provides a meaningful error message to the client. 

``response_attribute_filter_template`` and ``request_attribute_filter`` can be used to make exceptions to the parsing and serializing rules, this is discussed in :ref:`exceptions-to-the-rule`.

GET requests however will have option of providing a combination of name, value pairs that match any or none of these sets. Parameter Set matching is forgiving, your GET handler will be executed regardless of the result of the matching process. Parameter Sets work on the *first in, best dressed** rule, the first set that matches the request satisfies the validation process.

.. note:: All parameters accepted in RequestParsers are instances.

By default all validation rules are set to ``None``, this tells prestans to ignore validation.

Attaching a request handler is as simple as assigning an instance of your ``RequestParser`` to your ``RESTRequestHandler``::

    class MyRESTRequestHandler(prestans.handlers.RESTRequestHandler):
        
        request_parser = MyRequestParser()

Following this prestans will execute the associated method to the request in your handler. 

You can reuse your ``RequestParser`` across multiple handlers. If your handler does not implement a particular method, any defined rules for that method will be ignored by prestans.

Parameter Sets
==============

REST handlers accept defined sets of URL parameters to allow the client to configure that way it responds. A popular use case is accepting values like `offset`, and `limit` which tells the handler the number of results to send down the wire.

Like ``Models`` prestans provides a well defined pattern to describe sets of Parameters (called ParameterSets), a set of which can be associated to a handler method. prestans evaluates parameters sent as part of the URL and attempts to match them to the provided templates. If a set of parameters match, they are made available as an instance of your ParameterSet subclass.

The request handler can access the parsed parameter set using ``self.request.parameter_set``. By default this is set to None.

``ParameterSets`` are matched on a first come best dressed principal. If you find that yourself defining sets that with one too many overlapping instances variables, you might want to re-think the design of your API call.

ParameterSets are defined by sub-classing ``prestans.parsers.ParameterSet``. Since data provided in a URL are name value pairs, prestans only allows the use of basic types `(String, Integer, Float)` in ParameterSets.

Consider the following two ParameterSet definitions, one of them allows searching by Keywords, the other by an unread flag, both of them have the common parameters ``offset`` and ``limit``.

.. code-block:: python

    class KeywordSearchParameterSet(prestans.parsers.ParameterSet):

        keyword = prestans.types.String(required=True)
        offset = prestans.types.Integer(required=False, default=0)
        limit = prestans.types.Integer(required=False, default=10)

    class UnreadParameterSet(prestans.parsers.ParameterSet):

        unread = prestans.types.Boolean(required=True, default=False)
        offset = prestans.types.Integer(required=False, default=0)
        limit = prestans.types.Integer(required=False, default=10)


Parameter Sets are defined in a handler method's ``ParserRuleSet`` which in turn is associated to the handler. prestans follows this design principle throughout the framework to ensure you can reuse as many definitions as possible across handlers in your application.

.. code-block:: python

    class MyRequestParser(prestans.parsers.RequestParser):

        GET = prestans.parsers.ParserRuleSet(
            parameter_sets = [
                KeywordSearchParameterSet(),
                UnreadParameterSet()
            ]
        )

If the client was to call the following URL (assuming you are running a local development server)::

    http://localhost/api/myhandler?keyword=something

this would result in prestans assigning an instance of ``KeywordSearchParameterSet`` to the request handler's ``self.request.parameter_set`` attribute with values from the URL request parsed as the expected types, and likewise for the ``UnreadParameterSet`` if the parameter unread was passed. Since neither requests provide the ``offset`` or ``limit`` parameters the default values would be assigned to the attributes.

If the client provides values that violates the validation rules defined by the ParameterSet, prestans will reject that request.

All raw URL parameters can be access using the ``set.request.get(key_name)`` method. This would make available any parameter that do not belong to Parameter Sets.

.. note:: Raw URL parameters are always strings, you will have to explicitly convert types.

Request Body
============

Clients accessing REST APIs are expected to send messages in an agreed serialization format. prestans supports a range of serialization methods and provides infrastructure for you to write your own. JSON is probably the most popular serialization format for Ajax Web applications.

.. note:: Our examples assume JSON as the serialization format in use.

Your handler can define strict rules using prestans Models for this incoming data. ``Models`` is one of prestans's major feature and is discussed in great detail in it's own dedicated section. Models in a prestans application can be use to parse and serialize strongly validated data.

This section focuses on how you can use Models to parse incoming data. Assume you have a very simple Model defined as follows::

    class Album(prestans.types.Model):

        title = prestans.types.String(required=True)
        release_year = prestans.types.Integer(required=True, min_value=1200, max_value=2012)
        genre = prestans.types.String(required=True, choices=['rock', 'blues', 'pop'])

your REST handler can use a ParserRuleSet to indicate that it wishes to use this model as the template for data sent via the request body. Remember that the serializer chosen as part of your URL router definition is responsible for unserializing the input before Model it's parsed. If unserialization fails prestans will reject the request. An example could look like::

    class MyRequestParser(prestans.parsers.RequestParser):

        POST = prestans.parsers.ParserRuleSet(
            body_template=Album(),
        )

If the body is successfully parsed, an instance of the Model class (with values parsed from the request) is assigned to ``self.request.parsed_body``. On failing to parse the body prestans will reject the request providing the client meaningful information about the failure.

.. _exceptions-to-the-rule:

Making exceptions to the rule
=============================

Keeping Request and Response sizes as small as possible is crucial for performance in REST application. Model design should be strict, to ensure the quality of the data accepted and delivered by your REST services. We pointed out earlier, that by default validation for request and response bodies is absolutely unforgiving.

There are times that you need to make an exception to the rule, consider the following scenarios:

* You have full text description in a Model which you do not want included in the default response. The client has to exclusively request the full text description
* In reverse you might want a service that a client can send only the textual description for update.

One of the ways you can handle this is by writing numerous models that each REST service uses, this works at first but for large applications you'll find yourself maintaining a one too many REST models. If you wish to use DataAdapters to build responses, you have to ensure that you register each defined model, and so on.

prestans offers an easy, clearly defined way per handler to make exceptions to the parsing rules while accepting requests or building responses. This is done assigning ``AttributeFilter`` instances to your ``ParserRuleSet`` or the handler's response.

``AttributeFilter`` objects are a dynamically configurable sets of rules that can be used in prestans. Each attribute can either have a ``Boolean`` or an instance of ``AttributeFilter`` as it's value. Assigning instances of ``AttributeFilter`` to attributes is how you create a sub filter.

.. code-block:: python

    my_attr_filter = prestans.parsers.AttributeFilter()
    my_attr_filter.name = True
    my_attr_filter.phone = True
    my_attr_filter.notes = False

    # Sub filter
    my_attr_filter.addresses = prestans.parsers.AttributeFilter()
    my_attr_filter.addresses.street_name = True
    my_attr_filter.addresses.city = True
    my_attr_filter.addresses.state = False

In most cases AttributeFilters are reflection of a Model, so AttributeFilter can be created directly from a model. Optionally you can set the default state of each attribute, by default this is set to False, hence all attributes will be hidden unless specified otherwise.

.. code-block:: python

    # Typical usage
    my_attr_filter = prestans.parsers.AttributeFilter.from_model(MyModel())

    # Usage if you want to override the default value
    my_attr_filter = prestans.parsers.AttributeFilter.from_model(MyModel(), default_value=True)

    # You can change the values after instantiation from a model
    my_attr_filter.notes = False

Once you've created a filter, all you have to do is tell prestans to use it while evaluating inbound requests or building responses. Here's how.

Request Attribute Filter
------------------------

.. code-block:: python

    my_attr_filter = prestans.parsers.AttributeFilter.from_model(MyModel(), default_value=True)
    my_attr_filter.notes = False

    class MyRequestParser(prestans.parsers.RequestParser):

        GET = prestans.parsers.ParserRuleSet(
            parameter_sets = [
                KeywordSearchParameterSet(),
                request_attribute_filter=my_attr_filter
            ]
        )


Providing a Response Attribute Filter Template
-----------------------------------------------

prestans allows clients to make sensible requests to cut down latency. Consider two very different use cases for your API, a business to business client and your traditional Web or Mobile client. They both care for very different sorts of data, one willing to wait longer than the other, process more data than the later.

Clients can ask prestans to modify the response by providing a JSON serialized configuration that an ``AttributeFilter``. This is provided as a parameter in the URL with the key ``_response_attribute_list``. This key is reserved by prestans and cannot be used by your application. 

.. code-block:: json

     { 
       field_name0: True, 
       field_name1: False, 
       collection_name0: True, 
       collection_name1: False,
       collection_name2: {
           sub_field_name0: True,
           sub_field_name1: False 
       }
     }

Your REST handler must provide a template prestans can match this input, if the JSON provided by the client has keys that are not present in the template, the request is rejected. 

.. code-block:: python

    class MyRequestParser(prestans.parsers.RequestParser):

        GET = prestans.parsers.ParserRuleSet(
            parameter_sets = [
                KeywordSearchParameterSet(),
                response_attribute_filter_template=prestans.parsers.AttributeFilter.from_model(MyModel())
            ]
        )

Your handler end point can get access to this ``AttributeFilter`` at ``self.response.attribute_filter``. Responses are filtered while prestans is serializing output. Keys of the object being serialized must match the attribute filter's list. If you are serializing models it's recommended you create your attribute filter using the model.

You can also manually set an ``AttributeFilter``, here an example of an ``AttributeFilter`` that turns the ``notes`` field off set inside the handler.

.. code-block:: python

    def get(self):

        ... do other stuff here first to build response

        # Create your attribute filter from your model
        my_attr_filter = prestans.parsers.AttributeFilter.from_model(MyModel())
        my_attr_filter.notes = False

        # Before you return assign it to self.response.attribute_filter
        self.response.attribute_filter = my_attr_filter

If an attribute in the filter is set to be hidden, the prestans serializer omits the key in the JSON response. While parsing on the client side, you should check for the existence of the key.