===================
Models (incomplete)
===================

Models allow you to define rules for your API's data. prestans uses these rules to ensure the integrity of the data exchanged between the client and the server. If you've' used the `Django <http://djangoproject.com>`_ or `Google AppEngine <https://developers.google.com/appengine/>`_ prestans models will look very familiar. prestans models are *not* persistent.

prestans types are one of the following:

* ``prestans.types.DataType`` all prestans types are a subclass of ``DataType``, this is the most basic DataType in the prestans world.
* ``prestans.types.DateStructure`` are a subclass of ``DataType`` but represent complex types like Date Time.
* ``prestans.types.DateCollection`` are a subclass of ``DataType`` and represent collections like Arrays or Dictionaries (refered to as Models in the prestans world).

Each type has configurable properties that prestans uses to validate data. It's important to design your models with the strictest case in mind. Use request and response filters to relax the rules for specific cases, refer to our chapter on :doc:`validation`.

This chapter introduces you to writing Models and using it in various parts of your prestans application. It is possible to write custom ``DataType``.

All prestans types are wrappers on Pythonic data types, that you get a chance to define strict rules for each attribute. These rules ensure that the data you exchange with a client is sane, ensures the integrity of your business logic and minimizes issues when persisting data. All of this happens even before your handler is even called.

*Most importantly* it cuts out the need for writing trivial boilerplate code to validate incoming and outgoing data. If your handler is called you can trust the data is sane and safe to use.

prestans types are divided into, *Basic Types*, and *Collections*, currently supported types are:

* ``String``, wraps a Python ``str``
* ``Integer``, wraps a Python number
* ``Float``, wraps a Python number
* ``Boolean``, wraps a Python ``bool``
* ``DataURLFile``, supports uploading files via HTML5 `FileReader <http://www.html5rocks.com/en/tutorials/file/dndfiles/>`_ API
* ``DateTime``, wraps Python ``datetime``
* ``Array``, wraps Python lists
* ``Model``, wraps Python dict

The second half of this chapter has a detailed reference of configuration parameters for each prestans ``DataType``.

Writing Models
==============

``Models`` are defined by extending ``prestans.types.Model``. Models contain attributes which either be a basic prestans type (a direct subclass of ``prestans.types.DataType``) or a reference to an instance of another Model, or an ``Array`` of objects.

The REST standard talks about URLs refering to entities, this is often interpreted literally as REST API URLs refer to persistent models. Your REST API is the *business logic* layer of your Web client / server application. Providing direct access to persistently stored data through your REST API is simply replicating XML-RPC and not only is it bad design in the RESTful world but also extremely insecure.

RESTful APIs should serve back REST models. REST models are views of your data, that make sense as a response to the REST request. It's important to understand this so you can define your REST models to be as strict as possible. A RESTful API should never accept a request it can't comply with, this includes authority to perform the requested tasks on the data.

Consider a scenario where we are trying to model discographies, where a ``Band`` has ``Albums``, has ``Tracks``.

Depending on the implementation of this applicaiton it might be easier to send down ``Tracks`` when a client requests ``Albums``, but might only want to send down ``Albums`` (without ``Tracks``) when a list of ``Bands`` is requested.

.. note:: Read our section of Design Notes, to learn more about designing better REST APIs.

General convention for prestans apps is to keep all your REST models in a single package. To start creating models, simply define a class that extends from ``prestans.types.Model``

.. code-block:: python

    ... amogst other things
    import prestans.types

    class Track(prestans.types.Model):

        ... next read about attributes here


Defining Attributes
-------------------

All atributes of a ``Model`` must be an instance of a ``prestans.type``, Attributes can also be relationships to instances or collections of Models.

Our :ref:`type-config-reference` guide documents in detail configuration validation options provided by each prestans ``DataType``.

Attributes are defined at a class level, these are the rules used by prestans for each instance attributes of your ``Model``. By default prestans is absolutely unforgiving and will ensure that each attribute satifies all it's conditions. Failure results in aborting the creation of an instance.

At the class level define attributes by instantiating prestans types with your rules, ensure they are as strict as possible, the more your define here the less you have to do in your handler. The objective is not to pass through data that your handler can't work with.

.. code-block:: python

    class Track(prestans.types.Model):

        id = prestans.types.Integer(required=False)
        name = prestans.types.String(required=True, min_length=1)
        duration = prestans.types.Float(required=True)


To One Relationship
-------------------

One to One relationships are supported 

.. code-block:: python

    class Band(prestans.types.Model):

        ... other attributes ...

        created_by = UserProfile()


To Many Relationship (using Arrays)
-----------------------------------

prestans provides ``prestans.types.Array`` to provide lists of objects. Collections in REST responses or requests must have elements of the same type. 

The ``element_template`` 

.. code-block:: python

    class Album(prestans.types.Model):

        ... other attributes ...

        tracks = prestans.types.Array(element_template=Track(), min_length=1)


Self References
---------------


.. code-block:: python

    class UserProfile(prestans.types.Model):

        id = prestans.types.Integer(required=False)
        email_address = prestans.types.String(required=True)

        name = prestans.types.String(required=True)

    class Track(prestans.types.Model):

        id = prestans.types.Integer(required=False)
        name = prestans.types.String(required=True, min_length=1)
        duration = prestans.types.Float(required=True)

        created_by = UserProfile()

    class Album(prestans.types.Model):

        id = prestans.types.Integer(required=False)
        name = prestans.types.String(required=True, min_length=1, default=prestans.types.CONSTANT.DATETIME_NOW)
        year = prestans.types.Integer(required=True)

        tracks = prestans.types.Array(element_template=Track(), min_length=1)

        created_by = UserProfile()

    class Band(prestans.types.Model):

        id = prestans.types.Integer(required=False)
        name = prestans.types.String(required=True, min_length=1)

        albums = prestans.types.Array(element_template=Album())

        created_by = UserProfile()


Special Types
-------------

Date Time
=========


DataURLFile
===========

Using Models to write Responses
-------------------------------



Using Data Adapters to build Responses
======================================

Pairing REST models to persistent models
----------------------------------------

Adapting Models
---------------

.. _type-config-reference:

Type Configuration Reference
============================

Basic prestans types extend from ``prestans.types.DataType``, these are the building blocks of all data represented in systems, e.g Strings, Numbers, Booleans, Date and Times.

Collections contain a series of attributes of both Basic and Collection types.

String
------

Strings are wrappers on Pythonic strings, the rules allow pattern matching and validation.

.. note:: Extends ``prestans.types.DataType``

* ``required`` flags if this is a mandatory field, accepts ``True`` or ``False`` and is set to ``True`` by default
* ``default`` specifies the value to be assigned to the attribute if one isn't provided on instantiation, this must be a String.
* ``min_length`` the minimum acceptable length of the String, if using the ``default`` parameter ensure it respects the length. 
* ``max_length`` the maximum acceptable length of the String, if using the ``default`` parameter ensure it respects the length.
* ``format`` a regular expression for custom validation of the String.
* ``choices`` a list of Strings that are acceptable values for the attribute.
* ``utf_encoding`` set to ``utf-8`` by default is the confiurable UTF encoding setting for the String.

Integer
-------

Integers are wrappers on Python numbers, limited to Integers. We distinguish between Integers and Floats because of formatting requirements.

.. note:: Extends ``prestans.types.DataType``

* ``required`` flags if this is a mandatory field, accepts ``True`` or ``False`` and is set to ``True`` by default
* ``default`` specifies the value to be assigned to the attribute if one isn't provided on instantiation, this must be a Integer.
* ``minimum`` the minimum acceptable value for the Integer, if using default ensure it's greater or equal to than the minimum.
* ``maximum`` the maximum acceptable value for the Integer, if using default ensure it's less or equal to than the maximum.
* ``choices`` a list of choices that the Integer value can be set to, if using default ensure the value is set to of the choices.

Float
-----

Floats are wrappers on Python numbers, expanded to Floats.

.. note:: Extends ``prestans.types.DataType``

* ``required`` flags if this is a mandatory field, accepts ``True`` or ``False`` and is set to ``True`` by default
* ``default`` specifies the value to be assigned to the attribute if one isn't provided on instantiation, this must be a Float.
* ``minimum`` the minimum acceptable value for the Float, if using default ensure it's greater or equal to than the minimum.
* ``maximum`` the maximum acceptable value for the Float, if using default ensure it's less or equal to than the maximum.
* ``choices`` a list of choices that the Float value can be set to, if using default ensure the value is set to of the choices.


Boolean
-------

Booleans are wrappers on Python ``bools``.

.. note:: Extends ``prestans.types.DataType``

* ``required`` flags if this is a mandatory field, accepts ``True`` or ``False`` and is set to ``True`` by default
* ``default`` specifies the value to be assigned to the attribute if one isn't provided on instantiation, this must be a Boolean.

DataURLFile
-----------

Supports uploading files using the HTML5 `FileReader <http://www.html5rocks.com/en/tutorials/file/dndfiles/>`_ API.

.. note:: Extends ``prestans.types.DataType``

* ``required`` flags if this is a mandatory field, accepts ``True`` or ``False`` and is set to ``True`` by default
* ``allowed_mime_types``

DateTime
--------

Date Time is a complex structure that parses strings to Python ``datetime`` and vice versa. Default string format is ``%Y-%m-%d %H:%M:%S`` to assist with parsing on the client side using Google Closure Library provided `DateTime <http://closure-library.googlecode.com/svn/docs/class_goog_date_DateTime.html>`_.

.. note:: Extends ``prestans.types.DataStructure``

* ``required`` flags if this is a mandatory field, accepts ``True`` or ``False`` and is set to ``True`` by default
* ``default`` specifies the value to be assigned to the attribute if one isn't provided on instantiation, this must be a date. prestans provides a constans ``prestans.types.CONSTRANT.DATETIME_NOW`` if you want to use the date / time of execusion.
* ``format`` default format  ``%Y-%m-%d %H:%M:%S``

Collections
===========

Collections are formalised representations to complex itterable data structures. prestans provides two Collections, Arrays and Models (dictionaries).

Array
-----

Arrays are collections of any prestans type. To ensure the integrity of RESTful responses, ``Array`` elements must always be of the same kind, this is defined by specifying an ``element_template``. prestans Arrays are itterable.

.. note:: Extends ``prestans.types.DataCollection``

* ``required`` flags if this is a mandatory field, accepts ``True`` or ``False`` and is set to ``True`` by default
* ``default`` a default object of type ``prestans.types.Array`` to be used if a value is not provided
* ``element_template`` a instance of a ``prestans.types`` subclass that's use to validate each element. prestans does not allow arrays of mixed types because it does not form valid URL responses.
* ``min_length`` minimum length of an array, if using default it must conform to this constraint
* ``max_length`` maximum length of an array, 

Model
-----

Models are wrapper on dictionaries, it provides a list of key, value pairs formalised as a Python ``class`` made up of any number of prestans ``DataType`` attributes. Models can have instances of other models or Arrays of Basic or Complex prestans types.

.. note:: Extends ``prestans.types.DataCollection``

* ``required`` flags if this is a mandatory field, accepts ``True`` or ``False`` and is set to ``True`` by default
* ``default`` a default model instance, this is useful when defining relationships
* ``**kwargs`` a set of key value arguments, each one of these must be an acceptable value for instance variables, all defined validation rules apply.
