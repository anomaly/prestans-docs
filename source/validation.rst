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

.. note: All parameters accepted in RequestParsers are instances.

By default all validation rules are set to ``None``, this tells prestans to ignore validation.

Attaching a request handler is as simple as assigning an instance of your ``RequestParser`` to your ``RESTRequestHandler``::

    class MyRESTRequestHandler(prestans.handlers.RESTRequestHandler):
        
        request_parser = MyRequestParser()

Following this prestans will execute the associated method to the request in your handler. 

You can reuse your ``RequestParser`` across multiple handlers. If your handler does not implement a particular method, any defined rules for that method will be ignored by prestans.

Parameter Sets
==============

``self.request.parameter_set``


Request Body
============

``self.request.parsed_body``

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