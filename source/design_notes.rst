======================
Thoughts on API design
======================

prestans was a result of our careful study into the REST standards, frameworks and appraoches that were popular at the time. The following are a few useful lessons we've learnt along the way. Also refer to our extensive list of extremely useful :doc:`reference_material` we found on the Web.

REST resources are *not* persistent models
==========================================

Reading around the Web, it seems that traditional client/server programmers somehow concluded that REST is basically a HTTP replacement for XML-RPC, SOAP lovers might have had something to do with this as well. This school of thought lead developers to design of REST APIs (like XML-RPC) as a gateway to each persistent object on the server and making the client responsible for dealing with data relationships, integrity etc. Many frameworks took these ideas and implemented pass through REST gateways to RDBMS backends.

This is completely incorrect.

Data presented to clients talking to REST services is very different to the way data is stored, this is particularly true when you are using NoSQL style databases. **Think of REST resources are views of the stored data**. The job of your server side code to do as much meaningful work as possible with the data and present it to the client in form that is immediately useful.

Again, *REST resources are useful views of your persistent data*.

Collections & Entities
======================

URLs should refer to resource or a kind of data that your client can work with. Resources are *not* persistent entities rather a view of them. There generally are two patterns for each resource that you need to address. Consider the following URL patterns

* /api/product
* /api/product/{id} 

Both deal with a resource called product. The first URL deals with collections, so get all products (GET), or create a new product (POST) are the requests it should respond to. 

The second would deal with a specific entity of that kind of resource. So get a product (GET), Update a product (PUT, PATCH), or delete a product (DELETE) are the requests it should respond to.

As a design principle we recommend you handle collections and entities in two seaprate handlers.

Response Size does matter
=========================

Database, Web Servers, prestans your handlers, servers are generally pretty quick (if you have written most things well). Network latency is still a killer for REST applications. 

A general view is that latency is generally caused by services on the server side running slow, althought can be the case, one thing that slips out of the radar is the size of the response that you send down to the client.

One of our latest applications was sending down large amounts of textual data, was never a problem when were building the application but as it was put to the production the size of stored text went out of hand, pushing the size of a 100 record response to 2.5 Megabytes. It wasn't MySQL, wasn't our code, prestans, Apache, or the server it was purely the size of response.

So when writing REST services, **Size really does matter!**
