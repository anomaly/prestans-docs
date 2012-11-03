======
Models
======

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

* String, wraps a Python str
* Integer, wraps a Python number
* Float, wraps a Python number
* Boolean, wraps a Python bool
* DataURLFile, supports uploading files via HTML5 `FileReader <http://www.html5rocks.com/en/tutorials/file/dndfiles/>`_ API
* DateTime, wraps Python ``datetime``
* Array, wraps Python lists
* Model, wraps Python dict

The second half of this chapter has a detailed reference of configuration parameters for each prestans ``DataType``.

Writing Models
==============

It's important to keep in mind that prestans or REST models are not persistent and are nearly never a direct translation of your persistent models. Clients require views of the data stored on the server.

.. code-block: python


To One Relationship
-------------------

To Many Relationship (using Arrays)
-----------------------------------

Using Models to write Responses
-------------------------------

Using Data Adapters to build Responses
======================================

Pairing REST models to persistent models
----------------------------------------

Adapting Models
---------------

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
* ``default`` specifies the value to be assigned to the attribute if one isn't provided on instantiation, this must be a date.
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
