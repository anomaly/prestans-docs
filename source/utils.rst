=========
Utilities
=========

This section covers a set of utilities shipped with prestans. These features are complimentary to API design but are not required for your application to function.

prestans.util.autocaste
=======================

``@prestans.util.autocaste`` is a decorator that automatically castes each incoming parameter per handler to their right type. ``prestans.util.autocaste`` takes positional arguments of Python types that must match your handler signature.

.. code-block:: python

    @prestans.util.autocaste(int, int)
    def get(self, band_id, album_id):
        ... do what you need here

We decided to take this approach to auto casting because it's a per handler and environment specific decision. Our solution is designed to assist not assert.


API Blueprint
=============

prestans ships with a built-in handler that can produce a blueprint for your prestans API. Documentation is a developer's nightmare, mostly because it's difficult to think back and encapsulate all parts of your design (not to mention that it's not the most exciting part of the job). However it's one of the most important ingredients of success. Consumers are most interested in endpoints provided by your API and what each endpoint expects, e.g. data payloads, URL parameters, etc. 

prestans ships with an inbuilt handler that inspects all handlers, models, parameter sets, attribute filters and makes available a description in your chosen serialization format.

.. code-block:: python

    import prestans.rest

    # Provides BlueprintHandler
    import prestans.handlers

    import pdemo.handlers
    import pdemo.rest.handlers.album
    import pdemo.rest.handlers.band
    import pdemo.rest.handlers.track

    api = prestans.rest.JSONRESTApplication(url_handler_map=[

        # Add the blueprint handler to /api/blueprint
        (r'/api/blueprint', prestans.handlers.BlueprintHandler),

        # Application handlers
        (r'/api/band', pdemo.rest.handlers.band.Collection),
        (r'/api/band/([0-9]+)', pdemo.rest.handlers.band.Entity),
        (r'/api/band/([0-9]+)/album', pdemo.rest.handlers.album.Collection),
        (r'/api/band/([0-9]+)/album/([0-9]+)/track', pdemo.rest.handlers.track.Collection)

    ], application_name="prestans-demo", debug=False)


.. note:: If you are planning to make blueprints available on your live service, we seriously recommend using a caching mechanism. Blueprints introspect every handler, parameter set, model to produce it's output and could prove to be computationally expensive.

