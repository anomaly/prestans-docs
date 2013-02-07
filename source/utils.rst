=========
Utilities
=========

This section covers a set of utilities shipped with prestans. These features are complimentary to API design but are not required for your application to function.

prestans.util.signature
=======================

``@prestans.util.signature`` is a decorator that automatically castes each incoming parameter per handler to their right type. ``prestans.util.signature`` takes positional arguments of Python types that must match your handler signature.

.. code-block:: python

    @prestans.util.signature(self, int, int)
    def get(self, band_id, album_id):
        ... do what you need here

We decided to take this approach to variable casting because it's a per handler and environment specific decision. Our solution is designed to assist not assert.

.. note:: This function is based upon `Andrew Lee <http://stackoverflow.com/users/586660/andrew-lee>`_'s blog post `Faux function type signatures in Python <http://www.regularexpressionless.com/?p=8>`_


API Blueprint
=============

prestans ships with a special built-in handler base that can produce a blueprint for your prestans API. Documentation is a developer's nightmare, mostly because it's difficult to think back and encapsulate all parts of your design (not to mention that it's not the most exciting part of the job). However it's one of the most important ingredients for success. Consumers are most interested in endpoints provided by your API and what each endpoint expects, e.g. data payloads, URL parameters, etc. 

prestans ships with an inbuilt handler that inspects all of your application's registered handlers, models, parameter sets, attribute filters and makes available a description in your chosen serialization format.

Presumeably you might to expose the blueprint to the public, and so you can leverage from all the other features of prestans (e.g authentication, throttling, caching) prestans requires you to implement a simple handler that: 

* extends from ``prestans.handlers.BlueprintHandler``
* implements the method to respond to an ``GET`` request (all other requests to blueprint handlers are supressed)
* calls the ``create_blueprint`` method which returns a serializable dictionary
* add the returned dictionary to the response

A sample implementation would look something like this:

.. code-block:: python

    import prestans.handlers

    class APIBlueprintHandler(prestans.handlers.BlueprintHandler):

        def get(self):

            blueprint = self.create_blueprint()

            self.response.status_code = prestans.rest.STATUS.OK
            self.response.set_body_attribute("api", blueprint)

then map it as you would any other handler to a URL that you see fit, remember that this handler will be ignored from the API blueprint:

.. code-block:: python

    import prestans.rest

    import pdemo.handlers
    import pdemo.rest.handlers.album
    import pdemo.rest.handlers.band
    import pdemo.rest.handlers.track

    api = prestans.rest.JSONRESTApplication(url_handler_map=[

        # Add the blueprint handler to /api/blueprint
        (r'/api/blueprint', pdemo.rest.handlers.APIBlueprintHandler),

        # Application handlers
        (r'/api/band', pdemo.rest.handlers.band.Collection),
        (r'/api/band/([0-9]+)', pdemo.rest.handlers.band.Entity),
        (r'/api/band/([0-9]+)/album', pdemo.rest.handlers.album.Collection),
        (r'/api/band/([0-9]+)/album/([0-9]+)/track', pdemo.rest.handlers.track.Collection)

    ], application_name="prestans-demo", debug=False)


.. warning:: If you are planning to make blueprints available on your live service, we seriously recommend using a caching mechanism. Blueprints introspect every handler, parameter set, model to produce it's output and could prove to be computationally expensive.

Each auto generated blueprint:

* Is grouped by Python package that contains your handlers, each module is the key in a dictionary.
* Uses Python docstrings (`PEP 257 <http://www.python.org/dev/peps/pep-0257/>`_) to fetch descriptions on each handler class and method.
* Includes information on supported handler methods, Parameter Sets, Models, Attribute Filters, constraints of each attribute.
