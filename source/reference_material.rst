==================
Reference Material
==================

We found the following references useful while writing prestans, they cover a variety of advanced Python programming and Web development topics.

It's important that you understand the basic concepts of Python Web Programming. All our documentation and support is based around the assumption that you are familiar with Python Web development using WSGI and are writing Ajax Web apps.

WSGI
====

* `WSGI <http://www.wsgi.org/en/latest/index.html>`_ - the way Web servers talk to Python apps.
* `ReUsable Web Components with Python and Future Python Web <http://www.youtube.com/watch?v=Ui-mSFuUZmQ>`_ - presented by Ben Bangert (YouTube).
* `Hosting Python Web Applications <http://www.youtube.com/watch?v=PWIvm-uloMg>`_ - presented by Graham P Dumpleton, author of `mod_wsgi <http://modwsgi.googlecode.com>`_ (YouTube).

HTTP
====

* `Unacceptable Browser HTTP Accept Headers <http://www.gethifi.com/blog/browser-rest-http-accept-headers>`_ - `Kris Jordan <http://www.gethifi.com/authors/kris-jordan>`_'s comprehensive article on how different browser interpret the Accept header.
* `The Accept Header <http://shiflett.org/blog/2011/may/the-accept-header>`_ - `Chris Shifflett <http://shiflett.org/about>`_ describes the Accept Header.
* `A Beginner's Guide to HTTP Cache Headers <http://www.mobify.com/blog/beginners-guide-to-http-cache-headers/>`_ - Kyle Young's excellent description of HTTP cache headers.

Advanced Python
===============

* `Python WSGI reference implementation <http://docs.python.org/2/library/wsgiref.html>`_ - available in Python 2.5
* `Effbot <http://effbot.org/pyfaq/>`_ - (A Semi-Official) Python FAQ Zone
* `Python Decorators <http://www.python.org/dev/peps/pep-0318/>`_ - various prestans utilities are provided as decorators
* `Python Types and Objects <http://www.cafepy.com/article/python_types_and_objects/python_types_and_objects.html>`_ - an excellent article by Shalabh Chaturvedi on how Python sees Objects and Types.
* `Python Attributes and Methods <http://www.cafepy.com/article/python_attributes_and_methods/>`_ - another excellent article by Shalabh Chaturvedi providing an indepth understanding of how attributes and methods work.
* `Python Regular Expressions <https://developers.google.com/edu/python/regular-expressions>`_ - Google Developer article on regular expressions.
* `Regular Expressions by Example <http://flockhart.virtualave.net/RBIF0100/regexp.html>`_ - specific regular expression examples. 
* `Inspecting live objects in Python <http://www.doughellmann.com/PyMOTW/inspect/>`_ - the inspect module provides functions for introspecting on live objects and their source code. This article by Doug Hellmann shows off many really nice features like discovering method signatures, extracting docstrings, etc.
* `An Intro to logging <http://www.blog.pythonlibrary.org/2012/08/02/python-101-an-intro-to-logging/>`_ - learn about how to use and extend the Python logging feature.
* `Python Style Guide <http://www.python.org/dev/peps/pep-0008/#package-and-module-names>`_ - PEP 0008 standards on naming stuff in Python

Implementation specific posts:

* `Faux function type signatures in Python <http://www.regularexpressionless.com/?p=8>`_ - using a Python decorator to ensure that your functions get values in the right type from WSGI calls. Originally posted as a `response <http://stackoverflow.com/questions/7019283/automatically-type-cast-parameters-in-python>`_ on Stackoverflow.

Frameworks:

* `Another Do-It-Yourself Framework <http://docs.webob.org/en/latest/do-it-yourself.html>`_

Serialization format
====================

* `YAML ain't a markup language <http://jessenoller.com/blog/2009/04/13/yaml-aint-markup-language-completely-different>`_ - `Jess Noller <https://twitter.com/jessenoller>`_ talks about YAML.

Server Software
===============

* `Google App Engine <https://developers.google.com/appengine/>`_ - an extemely easy to work with Cloud platform run by Google.
* `mod_wsgi <http://code.google.com/p/modwsgi/>`_ - a connector module allowing your to run WSGI apps with Apache Web server.
* `wsgid <http://wsgid.com/>`_ - Wsgid is a generic WSGI handler for mongrel2 web server. Mongrel2 is a non-blocking web server backed by a high performance queue (0mq). Wsgid plays a gateway role between mongrel2 and your WSGI application, offering a full daemon environment with start/stop/reload functionality. 
* `Werkzeug <http://werkzeug.pocoo.org>`_ - The Python WSGI Utility Library (including a development HTTP server)
* `MongoDB <http://www.mongodb.org/>`_ - MongoDB (from "humongous") is a scalable, high-performance, open source NoSQL database. Written in C++.

Developer Tools
===============

* `JSON Lint <http://jsonlint.org>`_ - a hosted JSON validation service
* `JSON View <http://jsonview.com>`_ - a in browser JSON prettifier for Chrome and Firefox.
* `Postman <http://www.getpostman.com>`_ - a Chrome plugin to ease API testing.