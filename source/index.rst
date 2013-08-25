.. prestans documentation master file, created by
   sphinx-quickstart on Sat Aug 11 13:23:11 2012.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

prestans, a Python REST micro-framework
=======================================

.. warning:: This branch of the documentation is under agressive development along with the framework.

prestans is a WSGI (`PEP-333 <http://www.python.org/dev/peps/pep-0333/>`_) complaint micro-framework that allows you to rapidly build quality REST services by introducing a pattern of models, parsers and handlers and in turn taking care of boilerplate code. prestans is aimed towards turly Ajax applications where the client side is completely written in JavaScript using toolkits like `Google Closure <https://developers.google.com/closure/>`_.

prestans is currently hosted on `Github <http://github.com/prestans>`_ and distributed under the terms defined by the `New BSD license <http://opensource.org/licenses/bsd-license.php>`_. A list of current downloads is `available here <https://github.com/prestans/prestans/tags>`_.  We highly recommend using `PyPI <http://pypi.python.org>`_ to install prestans.

.. toctree::
   :numbered:
   :maxdepth: 3

   install
   serializer_deserializer
   handlers
   validation
   models
   exceptions
   client
   design_notes
   reference_material

Getting Help
------------

We encourge the use of our mailing lists (run on Google Groups) as the primary method of getting help. You can also write the developers through contact information `our website <http://etk.com.au>`_.

* `Discuss <http://groups.google.com/group/prestans-discuss>`_ general discussion, help, suggest a new feature.
* `Announcements <http://groups.google.com/group/prestans-announce>`_ security / release announcements.

Reporting Issues
^^^^^^^^^^^^^^^^

We prefer the use of our `Issue Tracker <https://code.google.com/p/prestans/issues/list>`_ on Google Code, to triage feature requests, bug reports. 

Before you lodge a lodge a ticket:

* Ensure that you ask a question on our list, there might already be answer out there or we might have already acknowledged the issue
* Seek wisdom from our beautifully written documentation 
* Google to see that it's not something to do with your server environment (versions of Web server, WSGI connectors, etc)
* Check to ensure that you are *not* lodging a duplicate request.

When reporting issues:

* Include as much detail as you can about your environment (e.g Server OS, Web Server and it's Version, WSGI connector)
* Steps that we can use to replicate the bug
* Share a bit of your application code with us, it goes a long way to replicate issues
