==========
Extensions
==========

prestans extensions are purpose built extensions that act as bridges between prestans elements and for argument sake persistent backends. Etensions build on the core prestans framework and are heavily dependent on environment specific packages.

Data Adapters
=============

Our :doc:`models` chapter discusses in detail, the use of Models to validate and build responses returned by handlers. Models can use AttributeFilters to make exceptions to the validation rules set out by your Model's original definition.

We identified the scenario and data validation benefits of converting persistently stored data to REST models, and in turn identified that it's a code laborious process.

DataAdapters fills that gap in prestans, it automates the process of converting persistent models into REST models by providing:

* A static registry ``prestans.ext.data.adapters.registry``, that maps persistent models to REST models
* QueryResultsIterator, that iterates through collections of persistent results and turns them into REST models. QueryResultsIterator is specific to backends and uses the registry to determine relationships between persistent and REST models. 

.. note:: You can map multiple REST models to the same persistent model.

For our sample code assume that rest models live in the ``pdemo.rest.models`` and the persistent models live in ``pdemo.models``, and is written for AppEngine.

prestans supports ``SQLAlchemy`` and AppEngine's ``ndb`` and ``datastore``. You can write your DataAdapter to support custom backends.

Pairing REST models to persistent models
----------------------------------------

The registry allows you to provide a map acceptable translations between persistent and REST models. If a persistent model maps to more than one REST model, DataAdapters try and make the sensible choice unless you explicitly provide the REST model you wish to adapt the data to.

General practice is to register the persistent models along side their definition. An excerpt from ``pdemo.models``.

Registering the persistent model is as easy as calling the ``register_adapter`` method on ``prestans.ext.data.adapters.registry``, and providing it an instance of the appropriate ``ModelAdapter``.

Consider a REST model defined ``prestans.rest.models``:

.. code-block:: python

    import prestans.types

    class Band(prestans.types.Model):

        id = prestans.types.Integer(required=False)
        name = prestans.types.String(required=True, max_length=30)

        albums = prestans.types.Array(element_template=Album(), required=False)


And then in your persistent model package, use ``prestans.ext.data.adapters.registry`` to join the dots. Ensure that all children models are present in the registry (e.g Album):

.. code-block:: python

    from google.appengine.ext import ndb
    from google.appengine.api import users

    import prestans.ext.data.adapters
    import prestans.ext.data.adapters.ndb

    class Band(ndb.Model):

        name = ndb.StringProperty()
        
        created = ndb.DateTimeProperty(auto_now_add=True)
        last_updated = ndb.DateTimeProperty(auto_now=True)
        
        @property
        def albums(self):
            return Album.query(ancestor=self.key).order(Album.year)

        @property
        def id(self):
            return self.key.id()

    # Register the persistent model to adapt to the Band rest model, also
    # ensure that Album is registered for the children models to adapt
    prestans.ext.data.adapters.registry.register_adapter(
        prestans.ext.data.adapters.ndb.ModelAdapter(
            rest_model_class=pdemo.rest.models.Band, 
            persistent_model_class=Band
        )
    )

Adapting Models
---------------

Once your models have been declared in the adapter registry, your REST handler:

* Query the data that your handler is expected to return
* Set the HTTP status code
* Use the appropriate QueryResultIterator to construct your REST adapted models
* Assign the returned collection to ``self.response.body``

.. code-block:: python

    from google.appengine.ext import ndb

    import pdemo.models
    import pdemo.rest.handlers
    import pdemo.rest.models

    import prestans.ext.data.adapters.ndb
    import prestans.handlers
    import prestans.parsers
    import prestans.rest

    class CollectionRequestParser(prestans.parsers.RequestParser):

        GET = prestans.parsers.ParserRuleSet(        
            response_attribute_filter_template=prestans.parsers.AttributeFilter.from_model(pdemo.rest.models.Band())
        )

    class BandCollection(pdemo.rest.handlers.Base):

        request_parser = CollectionRequestParser()

        def get(self):

            bands = pdemo.models.Band().query()
        
            self.response.http_status = prestans.rest.STATUS.OK
            self.response.body = prestans.ext.data.adapters.ndb.QueryResultIterator(
                collection=bands, 
                target_rest_instance=pdemo.rest.models.Band
            )

If you are using AttributeFilters (read our chapter on :doc:`validation` to learn how you can make exceptions to Model validation rules) you can pass them onto the QueryResultsIterator which results in the QueryResultsIterator skipping accessing that property all together significantly reducing the load on the Data Layer:

.. code-block:: python

    class BandCollection(pdemo.rest.handlers.Base):

        request_parser = CollectionRequestParser()

        def get(self):

            bands = pdemo.models.Band().query()
        
            self.response.http_status = prestans.rest.STATUS.OK
            self.response.body = prestans.ext.data.adapters.ndb.QueryResultIterator(
                collection=bands, 
                target_rest_instance=pdemo.rest.models.Band,
                attribute_filter = self.response.attribute_filter
            )


