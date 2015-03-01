Data Adapters
=============

The :doc:`validation` chapter demonstrates the use of Prestans ``Models`` to validate requests, build rules complaint responses and the use of ``AttributeFilters`` to make temporary exceptions to the validation rules.

``DataAdapters`` automate morphing persistent objects to Prestans models, it provides the following features:

* A static registry ``prestans.ext.data.adapters.registry``, that maps persistent models to REST models
* An instance convertor, used to convert an instance. Convertors are specific to backends and uses the registry to determine relationships between persistent and REST models. 
* A collection iterator, that iterates through collections of persistent results and turns them into REST models. It follows all the same rules as the instance convertor.

Out of the box Prestans supports:

* `SQLAlchemy <http://www.sqlalchemy.org/>`_ which in turn should allow you to support most popular RDBMS backends.
* AppEngine's `Python NDB <https://developers.google.com/appengine/docs/python/ndb/>`_ which is built on top of DataStore.

.. note:: REST services provide views of persistent data, DataAdapters allow you to map multiple REST models to the same persistent model.

For the purposes of this example lets assume our rest models live in the namespace ``musicdb.rest.models`` and the persistent models live in ``musicdb.models``, and is written for AppEngine.

Writing custom ``DataAdapters`` is quite straight forward. The Prestans project welcomes third party contributions.

Pairing REST models to persistent models
----------------------------------------

Before you can ask Prestans to convert persistent objects to REST model instances, you must use the ``registry`` you to pair persistent definitions to REST definitions. Prestans uses the REST model as the template for the data that will be transformed. While adapting data Prestans will:

* Inspect the REST model for a list of attributes
* Inspect the persistent model for the data
* Ensure that the data provided by the persistent model matches the rules definied by the REST model
* If the persistent model does not define an attribute, Prestans reverts to using the default value or ``None``
* If the value provided by the persistent model fails to validate, Prestans raises an ``prestans.exception.DataValidationException`` which will graceful respond to the requesting client.

If a persistent definition maps to more than one REST model defintion, DataAdapters will try and make the sensible choice unless you explicitly provide the REST model you wish to adapt the data to.

Registering the persistent model is done by calling the ``register_adapter`` method on ``prestans.ext.data.adapters.registry``, and an appropriate ``ModelAdapter`` instance.

Consider the following REST models defined in ``musicdb.rest.models``:

.. code-block:: python

    import prestans.types

    class Album(prestans.types.Model):

        id = prestans.types.Integer(required=False)
        name = prestans.types.String(required=True, max_length=30)        

    class Band(prestans.types.Model):

        id = prestans.types.Integer(required=False)
        name = prestans.types.String(required=True, max_length=30)

        albums = prestans.types.Array(element_template=Album(), required=False)


along with it's corresponding NDB persistent model defined in ``musicdb.models``:

.. code-block:: python

    from google.appengine.ext import ndb
    from google.appengine.api import users

    import prestans.ext.data.adapters
    import prestans.ext.data.adapters.ndb

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

.. note:: By convention we recommend the use of the namespace ``yourproject.rest.adapters`` to hold all your adapter registrations.

.. code-block:: python
    
    import musicdb.models
    import musicdb.rest.models


    # Register the persistent model to adapt to the Band rest model, also
    # ensure that Album is registered for the children models to adapt
    prestans.ext.data.adapters.registry.register_adapter(
        prestans.ext.data.adapters.ndb.ModelAdapter(
            rest_model_class=musicdb.rest.models.Band, 
            persistent_model_class=musicdb.models.Band
        )
    )

Adapting Models
---------------

Once your models have been declared in the adapter registry, your REST handler:

* Query the data that your handler is expected to return
* Set the appropriate HTTP status code
* Use the ``adapt_persistent_instance`` or ``adapt_persistent_collection`` from the appropriate package to transform your persitent objects to REST objects.
* Prestans will query the registry for any children objects that appear in the object it's attempt to adapt.
* Assign the returned collection to ``self.response.body`` to send a response to the client

Each ``DataAdapter`` provides two convenience methods:

* ``adapt_persistent_collection`` which iterates over a collection of persistent objects to a collection of REST models
* ``adapt_persistent_instance`` which iterates over an instance of a persistent object to a REST model

.. code-block:: python

    from google.appengine.ext import ndb

    import musicdb.models
    import musicdb.rest.handlers
    import musicdb.rest.models
    import musicdb.rest.adapters

    import prestans.ext.data.adapters.ndb
    import prestans.handlers
    import prestans.parsers
    import prestans.rest


    class BandCollection(musicdb.rest.handlers.Base):

        request_parser = CollectionRequestParser()

        def get(self):

            bands = musicdb.models.Band().query()
        
            self.response.http_status = prestans.rest.STATUS.OK
            self.response.body = prestans.ext.data.adapters.ndb.adapt_persistent_collection(
                collection=bands, 
                target_rest_instance=musicdb.rest.models.Band
            )

If you are using ``AttributeFilters``, you should pass the filter along to the adapter method enabling it to skip accessing that property all together. This can significantly reduce read stress on backends that support lazy loading properties:

.. code-block:: python

    # Collection of objects
    class BandCollection(musicdb.rest.handlers.Base):

        def get(self):

            bands = musicdb.models.Band().query()
        
            self.response.http_status = prestans.rest.STATUS.OK
            self.response.body = prestans.ext.data.adapters.ndb.adapt_persistent_collection(
                collection=bands, 
                target_rest_instance=musicdb.rest.models.Band,
                attribute_filter = self.response.attribute_filter
            )

    # Adapting a single instance
    class BandEntity(musicdb.rest.handlers.Base):

        def get(self, band_id):

            .. use the appropriate query to get the appropriate instance

            self.response.http_status = prestans.rest.STATUS.OK
            self.response.body = prestans.ext.data.adapters.ndb.adapt_persistent_instance(
                collection=band, 
                target_rest_instance=musicdb.rest.models.Band,
                attribute_filter = self.response.attribute_filter
            )

.. note:: Each handler has access to the approprite attribute filter at ``self.response.attribute_filter`` (see :doc:`validation`)

Writing your own DataAdapter
----------------------------

``DataAdapter`` can be easily extended to support custom backends. Writing an adapter for a custom backend involves providing:

* an ``adapt_persistent_collection`` method that iterates over a collection of persistent objects and transforms them into a collection of REST objects
* an ``adapt_persistent_instance`` method that converts a single instance of a persistent object to a REST object
* an implementation of a ``ModelAdapter`` class that implements ``adapt_persistent_to_rest`` method which converts a persistent object to a REST object. An instance of this is returned by the ``registry`` and is used by the convenience methods. This method detail how each property is transformed.

It's very likely that you will be able to reuse the code for the convenience methods from one of the ``DataAdapters`` shipped with Prestans. They are responsible for wrapping backend specific operations like accessing collection lengths.

A scaffold of the a custom ``DataAdapter`` looks as follows:

.. code-block:: python

    def adapt_persistent_instance(persistent_object, target_rest_class=None, attribute_filter=None):
        ... your custom implementation

    def adapt_persistent_collection(persistent_collection, target_rest_class=None, attribute_filter=None):
        ... your custom implementation


    class ModelAdapter(adapters.ModelAdapter):
           
        def adapt_persistent_to_rest(self, persistent_object, attribute_filter=None):
            ... your custom implementation here




