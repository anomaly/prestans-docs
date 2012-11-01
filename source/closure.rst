==========================
Closure Library Extensions
==========================

`Google Closure <https://developers.google.com/closure/library/>`_ is a set of JavaScript tools, that Google uses to build many of their core products. It provides:

* A `JavaScript Optimizer <https://developers.google.com/closure/compiler>`_ to build a distributable version of your application
* A comprehensive `JavaScript library <https://developers.google.com/closure/library>`_
* A `templating system <https://developers.google.com/closure/templates>`_ for JavaScript
* A `JavaScript style checker <https://developers.google.com/closure/utilities>`_ and style fixer
* An `enhanced stylesheet language <http://code.google.com/p/closure-stylesheets/>`_ that works with the optimizer to minifiy CSS.

Each one of these components is agnostic of the other. Closure is at the heart of building products with prestans.

prestans provides a number of extensions to Closure Library, that ease and automate building rich JavaScript clients that consume your prestans API. It currently provides:

* REST Client
* Types API
* Bound UI

REST Client
===========

Prestans contains a ready made REST Client to allow you to easily make requests and unpack responses from a prestans enabled server API.

Client
------

Request
-------

Response
--------

Types API
=========

The Types API is a client side implementation of the types that are available in the server side API.


Bound UI
========

The Bound UI is a system that allows you to bind your input forms to prestans models to have basic validation of data occur as the user is entering it.

