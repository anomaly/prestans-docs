===========================
Routing & Handling Requests
===========================

First order of business is mapping URLs back to your code. prestans comes with an inbuilt router to help you achieve this, the router is paired with a serializer and manages the lifecycle of the REST API request. Out of the box prestans provides support for:

* JSON
* YAML

We plan to support other formats as we need them. You can also write your own for formats you wish to support in your application. Read the section on :doc:`serializers` to learn more about how serailziers work and how you can write your own. Our examples assume the use of JSON as the serialization format.

Each serialzier paired with paired with an attribute


Lifecycle
=========


Routing Requests
================

* url_map
* application_name
* debug

Building your Response
======================

.. note:: Data Adapters help you quickly turn you persistent data to REST models instances.

