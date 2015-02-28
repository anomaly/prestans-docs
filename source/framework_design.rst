================
Framework Design
================

Prestans is a developer-to-developer product. Before we start talking about how you can start harnessing it's features, we thought it might be a good idea to introduce you to some of the design decisions and patterns. This chapter highlights some of under the hood design decisions, it's written so you can gain an understanding of why Prestans behaves the way it does.

It covers the use of HTTP headers to control features of Prestans, and when Prestans will decide to bail out processing a request and when it will choose to fail gracefully (still responding back to the requesting client).

If you trust that we have made all the right decisions and you'd rather start working on your Prestans powered API, then head straight to :doc:`handlers`

Exceptions
==========

Prestans raises two kinds of exceptions. It handles both of them in very different ways:

* The first are inbuilt Python exceptions e.g ``TypeError``, ``AssertionError``, Prestans does not handle these exceptions, causing your API to fail process that particular request. This is because these are only raised if you have incorrectly configured your Prestans handler, or more importantly your handler is not respecting it's own rules e.g objects returned by an end point don't match your parser configuration.

* The second are Prestans defined exceptions. These are handled by Prestans and the router will return a proper error response to the requesting client. These are raised if the client placed an incorrect request e.g An attribute wasn't the right length or type. These excpetions are defined in ``Prestans.exceptions``, along with a guide for it's suggested use.

Prestans defined exceptions can also be used by your handler to notify the client of trivial REST usecases e.g requested entity does not exists. We talk about these in :doc:`handlers`.

HTTP Header Reference
=====================

HTTP headers are components of the message header for both HTTP requests and responses. They define the rules (e.g preferred content, cookies, software version) for each HTTP requests. These assist the server to best respond to the request and ensures that the client is able to consume it properly.

Prestans uses a combination of standard and custom headers to allow the requesting client to control the behavior of the API (without you having to write any extra code). Some of these headers can be specific to an HTTP Verb (e.g GET requests don't have request bodies and shouldn't send headers specifying the mime type of the body).

Prestans' `client side <https://github.com/Prestans/Prestans-client/>`_ add-ons for `Google Closure <https://developers.google.com/closure/library/>`_ wrap these up properly client library calls.

Content Negotiation
-------------------

Prestans APIs can speak as many serialization formats as they please. We support JSON and Plist XML out of the box. It's possible to write custom serializers and deserializers if you wish to support any other formats. 

Each API request uses the following standard HTTP headers to negotiate the content type of the request and response payload:

* ``Accept`` - which tells the server formats that the client is willing to format, Prestans compares this to what the current API support, or sends back an error message using the default serializer.
* ``Content-Type`` - which tells the server what format the request body is; this is only relevant for certain HTTP Verbs. If the Prestans API does not support the format it sends back an error message using the default serializer.

.. note:: Prefixed URLs to separate content types is an *ugly* solution. If you are new to REST, we highly recommend watching `Michael Mahemoff <http://mahemoff.com>`_'s `Web Directions Code 2013 <http://code13.webdirections.org>`_ presentation on REST, `What every developer should know about REST <https://www.youtube.com/watch?v=2yAQ-yLq5eI>`_. 

Both these headers are part of the HTTP standards, the handler section in this chapter discusses how you pair up serializers and deserializers to these headers.


Custom headers
--------------

Prestans allows the requesting the client to control what parts of an entity they, they want returned as part of a response. The Prestans features we are referring to are (these are talked about in detail in other chapters):

* **Attribute Filters**, allows the client to toggle the visibility of attributes in a response, thus controlling the size and in turn the responsiveness of the API call. The typical use case of an attribute filter:
  
   * The client requests *only* the key and title of all blog posts
   * The UI waits for the user to choose a post they want to view
   * The client requests *only* the body of the blog post
   * The client displays the blog post in full
   * The client chooses to *cache* the body of that particular post by merging it into the partial entity

* **Minification**, Prestans REST models are designed to be descriptive to ensure your code reads well. Minification rewrites the attribute names in a handler response reducing the payload size. This is particularly useful when you are downloading collections (e.g list of blog posts). The response size can shrink by upto 30%.

Each Prestans may choose to override the requesting client's wishes. This is a design choice, and you must have a very ridig usecase for ignoring the client's request configuration.

Clients use a set of custom HTTP headers to configure these options. These are detailed as headers that a client sends to an API, and information the API sends back as HTTP headers. 

Your Web server environment will limit the size of a request and how large headers can be.

.. note:: IETF's `RFC6648 <http://www.ietf.org/rfc/rfc6648.txt>`_ deprecates the use the ``X-`` prefix for custom headers.

Inbound headers:

* ``Prestans-Version``, expected minimum version of the Prestans framework you're expecting to find on the server side. This ensures that your reuqest can reliably processed by the server.

* ``Prestans-Minification``, expects the string ``On`` or ``Off`` to toggle minification. This is set to ``Off`` by default.

* ``Prestans-Response-Attribute-List``, a serialized and nested structure of booleans to toggle the visibility of each attribute that you are expecting in the response. By default it's assumed that you wish to receive all attributes of the entity. If minification is turned on you must use the minified attribute names. Serialization format is controlled by the ``Content-Type`` and ``Accept`` header, this must be the same for the payload and the content of this header.

Outbound headers:

* ``Prestans-Version``, the version number of the Prestans framework that the server's running. This allows the client code to ensure that it's can process the response.
