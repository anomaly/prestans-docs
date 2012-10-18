======
Models
======

Models allow you to define rules for your API's data. prestans uses these rules to ensure the integrity of the data exchanged between the client and the server.

* ``prestans.types.DataType``
* ``prestans.types.DateStructure``
* ``prestans.types.DateCollection``


Supported Data Types
====================

Basic Types
-----------

String
^^^^^^

.. note:: Extends ``prestans.types.DateType``

* ``required``
* ``default``
* ``min_length``
* ``max_length``
* ``format``
* ``choices``
* ``utf_encoding``

Integer
^^^^^^^

.. note:: Extends ``prestans.types.DateType``

* ``default``
* ``minimum``
* ``maximum``
* ``required``
* ``choices``

Float
^^^^^

.. note:: Extends ``prestans.types.DateType``

* ``default``
* ``minimum``
* ``maximum``
* ``required``
* ``choices``


Boolean
^^^^^^^

.. note:: Extends ``prestans.types.DateType``

* ``required``
* ``default``
* ``required``

DataURLFile
^^^^^^^^^^^

.. note:: Extends ``prestans.types.DateType``

* ``required``
* ``allowed_mime_types``

DateTime
^^^^^^^^

.. note:: Extends ``prestans.types.DateStructure``

* ``required``
* ``default``
* ``format`` default format  ``%Y-%m-%d %H:%M:%S``

Collections
-----------

Array
^^^^^

.. note:: Extends ``prestans.types.DateCollection``

* ``default``
* ``required``
* ``element_template``
* ``min_length``
* ``max_length``

Model
^^^^^

.. note:: Extends ``prestans.types.DateCollection``

* ``required``
* ``default``
* ``**kwargs``


Writing Models
==============



Using Models to write Responses
-------------------------------

Using Data Adapters to build Responses
======================================