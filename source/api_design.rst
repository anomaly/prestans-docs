======================
Thoughts on API design
======================

Prestans was a result of our careful study into the REST standards, popular frameworks and appraoches. Following are useful lessons we've learnt along the way. We've also compiled an extensive list of extremely useful :doc:`reference_material` we found during our research.

REST resources are *not* persistent models
==========================================

The word entity traditionally referred to persistent objects. That concept applied on the Web results in a HTTP implementation of approaches like XML-RPC. Many frameworks implement REST as a gateway to directly access persistent object on the server.

Entity in the REST world refers to entities returned by REST service; not what's stored in the persistence layer. In most instances it refers to an application specific view of the persistent data; that the requesting client will find most useful.

REST endpoints form the business logic layer of your Ajax/Mobile application. REST services should do the heavy lifting and return the most useful view of the data for the use-case; this may include related data i.e Order may include Order line.

Server round trips (or latency) is one of biggest performance overheads for REST services.

Collections & Entities
======================

URLs should refer to resource or a kind of data that your client can work with. Resources are *not* persistent entities rather a view of them. There generally are two patterns for each resource that you need to address. Consider the following URL patterns

* ``/api/product``
* ``/api/product/{id}``

Both deal with a resource called product. The first URL deals with collections, so get all products (GET), or create a new product (POST) are the requests it should respond to. 

The second would deal with a specific entity of that kind of resource. So get a product (GET), Update a product (PUT, PATCH), or delete a product (DELETE) are the requests it should respond to.

As a design principle we recommend you handle collections and entities in separate handlers.

Response Size
=============

Running local development HTTP servers is common practice. Due to negligible latency; local server are notorious for masking server round trip issues, specially ones related to response size. Only when you've deployed your application to a remote server can you judge how long noticing how long round trips take.

Database, Web Servers, Prestans your handlers, servers are generally pretty quick (if you have written most things well). Network latency caused by the sheer size of the response can make your REST services appear to be slow. 

It's important not to loose sight of the response size written out by your REST services.