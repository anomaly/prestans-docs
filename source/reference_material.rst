==================
Reference Material
==================

We found the following references useful while writing prestans, they cover a variety of advanced Python programming and Web development topics.

It's important that you understand the basic concepts of Python Web Programming. All our documentation and support is based around the assumption that you are familiar with Python Web development using WSGI and are writing Ajax Web apps.

WSGI
====

* `WSGI <http://www.wsgi.org/en/latest/index.html>`_ the way Web servers talk to Python apps.
* `ReUsable Web Components with Python and Future Python Web <http://www.youtube.com/watch?v=Ui-mSFuUZmQ>`_ presented by Ben Bangert (YouTube).
* `Hosting Python Web Applications <http://www.youtube.com/watch?v=PWIvm-uloMg>`_ presented by Graham P Dumpleton, author of `mod_wsgi <http://modwsgi.googlecode.com>`_ (YouTube).

Advanced Python
===============

* `Python Decorators <http://www.python.org/dev/peps/pep-0318/>`_ various prestans utilities are provided as decorators
* `Python Types and Objects <http://www.cafepy.com/article/python_types_and_objects/python_types_and_objects.html>`_ an excellent article by Shalabh Chaturvedi on how Python sees Objects and Types.
* `Python Attributes and Methods <http://www.cafepy.com/article/python_attributes_and_methods/>`_ another excellent article by Shalabh Chaturvedi providing an indepth understanding of how attributes and methods work.
* `Faux function type signatures in Python <http://www.regularexpressionless.com/?p=8>`_ using a Python decorator to ensure that your functions get values in the right type from WSGI calls. Originall posted as a `response <http://stackoverflow.com/questions/7019283/automatically-type-cast-parameters-in-python>`_ on Stackoverflow. 
* `Inspecting live objects in Python <http://www.doughellmann.com/PyMOTW/inspect/>`_ the inspect module provides functions for introspecting on live objects and their source code. This article by Doug Hellmann shows off many really nice features like discovering method signatures, extracting docstrings, etc.

Software
========

* `Google App Engine <https://developers.google.com/appengine/>`_ an extemely easy to work with Cloud platform run by Google.
* `mod_wsgi <http://code.google.com/p/modwsgi/>`_, a connector module allowing your to run WSGI apps with Apache Web server.
* `wsgid <http://wsgid.com/>`_, Wsgid is a generic WSGI handler for mongrel2 web server. Mongrel2 is a non-blocking web server backed by a high performance queue (0mq). Wsgid plays a gateway role between mongrel2 and your WSGI application, offering a full daemon environment with start/stop/reload functionality. 
* `MongoDB <http://www.mongodb.org/>`_, MongoDB (from "humongous") is a scalable, high-performance, open source NoSQL database. Written in C++.

Developer Tools
===============

* `JSON Lint <http://jsonlint.org>`_, a hosted JSON validation service
* `JSON View <http://jsonview.com>`_, a in browser JSON prettifier for Chrome and Firefox.
* `Postman <http://www.getpostman.com>`_ a Chrome plugin to ease API testing.