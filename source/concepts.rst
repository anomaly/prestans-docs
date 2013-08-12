======================
Understanding prestans
======================

* Serializers
* Router
* Handler
* Models
* Parsers
* Data Adapters
* Providers


Serializers
===========

* Textual
* Binary

prestans HTTP Headers
=====================

Accept
Content-Type

For GET requests there are no Content-Type

Sent by the client:

* Prestans-Version
* Prestans-Response-Attribute-List
* Prestans-Response-Minification

Sent by the server:

* Prestans-Version
* Prestans-Rewrite-Map

Request lifecycle
==================

* Is the verb supported
* Is the requested mime type supported
* 
