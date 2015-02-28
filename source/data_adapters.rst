Data Adapters
=============

The :doc:`validation` chapter demonstrates the use of Prestans ``Models`` to validate requests, build rules complaint responses and the use of ``AttributeFilters`` are used to make temporary case by case exceptions to the validation rules

``DataAdapters`` automate morphing persistent objects to Prestans models, it provides the following feature

* A static registry ``Prestans.ext.data.adapters.registry``, that maps persistent models to REST models
* An instance convertor, used to convert an instance. Convertors are specific to backends and uses the registry to determine relationships between persistent and REST models. 
* A collection iterator, that iterates through collections of persistent results and turns them into REST models. It follows all the same rules as the instance convertor.

.. note:: REST services provide views of persistent data, DataAdapters allow you to map multiple REST models to the same persistent model.

For our sample code assume that rest models live in the ``musicdb.rest.models`` and the persistent models live in ``musicdb.models``, and is written for AppEngine.

Out of the box Prestans supports 

* `SQLAlchemy <http://www.sqlalchemy.org/>`_ which in turn should allow you to support most popular RDBMS backends.
* AppEngine's `Python NDB <https://developers.google.com/appengine/docs/python/ndb/>`_ which is built on top of DataStore.

Writing DataAdapters for other backends is discussed later in this chapter.

Pairing REST models to persistent models
----------------------------------------

Before you can ask Prestans to convert persistent objects to REST model instances, you must use the registry you to pair persistent definitions to REST definitions. Prestans uses the REST model as the template for the data that will be transformed. While adapting data Prestans will:

* Inspect the REST model for a list of attributes
* Inspect the persistent model for the data
* Ensure that the data provided by the persistent model matches the rules definied by the REST model
* If the persistent model does not define an attribute, Prestans reverts to using the default value or ``None``
* If the value provided by the persistent model fails to validate, Prestans raises an ``Prestans.exception.DataValidationException`` which will graceful respond to the requesting client.

If a persistent definition maps to more than one REST model defintion, DataAdapters will try and make the sensible choice unless you explicitly provide the REST model you wish to adapt the data to.

Registering the persistent model is done by calling the ``register_adapter`` method on ``Prestans.ext.data.adapters.registry``, and an appropriate ``ModelAdapter`` instance.

Consider the following REST models defined in ``musicdb.rest.models``:

.. code-block:: python

    import Prestans.types

    class Album(Prestans.types.Model):

        id = Prestans.types.Integer(required=False)
        name = Prestans.types.String(required=True, max_length=30)        

    class Band(Prestans.types.Model):

        id = Prestans.types.Integer(required=False)
        name = Prestans.types.String(required=True, max_length=30)

        albums = Prestans.types.Array(element_template=Album(), required=False)


along with it's corresponding NDB persistent model defined in ``musicdb.models``:

.. code-block:: python

    from google.appengine.ext import ndb
    from google.appengine.api import users

    import Prestans.ext.data.adapters
    import Prestans.ext.data.adapters.ndb

    class Album(ndb.Model):

        name = ndb.StringProperty()

        @property
        def id(self):
            return self.key.id()

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

We recommend that you create a package called ``yourproject.rest.adapters`` to hold all your adapter registrations. This is purely convention.

.. code-block:: python
    
    import musicdb.models
    import musicdb.rest.models


    # Register the persistent model to adapt to the Band rest model, also
    # ensure that Album is registered for the children models to adapt
    Prestans.ext.data.adapters.registry.register_adapter(
        Prestans.ext.data.adapters.ndb.ModelAdapter(
            rest_model_class=musicdb.rest.models.Band, 
            persistent_model_class=musicdb.models.Band
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

    import musicdb.models
    import musicdb.rest.handlers
    import musicdb.rest.models
    import musicdb.rest.adapters

    import Prestans.ext.data.adapters.ndb
    import Prestans.handlers
    import Prestans.parsers
    import Prestans.rest


    class BandCollection(musicdb.rest.handlers.Base):

        request_parser = CollectionRequestParser()

        def get(self):

            bands = musicdb.models.Band().query()
        
            self.response.http_status = Prestans.rest.STATUS.OK
            self.response.body = Prestans.ext.data.adapters.ndb.adapt_collection(
                collection=bands, 
                target_rest_instance=musicdb.rest.models.Band
            )

If you are using AttributeFilters, you should pass the filter along onto the ``adapt_collection`` method which allowing ``adapt_collection`` to skip accessing that property all together, this can significantly reduce read load on like NDB, or even SQLAlchemy if you lazy load relationships:

.. code-block:: python

    class BandCollection(musicdb.rest.handlers.Base):

        def get(self):

            bands = musicdb.models.Band().query()
        
            self.response.http_status = Prestans.rest.STATUS.OK
            self.response.body = Prestans.ext.data.adapters.ndb.QueryResultIterator(
                collection=bands, 
                target_rest_instance=musicdb.rest.models.Band,
                attribute_filter = self.response.attribute_filter
            )



Writing your own DataAdapter
----------------------------
