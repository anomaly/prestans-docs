.. prestans documentation master file, created by
   sphinx-quickstart on Sat Aug 11 13:23:11 2012.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

prestans, a Python REST micro-framework
=======================================

prestans is a WSGI (`PEP-333 <http://www.python.org/dev/peps/pep-0333/>`_) complaint micro-framework that allows you to rapidly build quality REST services by introducing a pattern of models, parsers and handlers and in turn taking care of boilerplate code. prestans is aimed towards turly Ajax applications where the client side is completely written in JavaScript using toolkits like `Google Closure <https://developers.google.com/closure/>`_.

prestans is currently hosted on `Google Code <http://prestans.googlecode.com>`_ and distributed under the terms defined by the `New BSD license <http://opensource.org/licenses/bsd-license.php>`_. A list of current downloads is `available here <https://code.google.com/p/prestans/downloads/list>`_.

.. toctree::
   :numbered:
   :maxdepth: 3

   quickstart
   handlers
   serializers
   validation
   models
   authentication
   ext
   design_notes
   closure
   demo_app
   reference_material


Software Requirements
---------------------

These are the requirements for running prestans on a server. Client side tools might have additional requirements, check relevant parts of our documentation for details.

.. note:: We use `Subversion <http://svn.tigris.org>`_ to manage our source code.

* Python 2.6+, *2.7 recommended*
* WSGI compliant server environment (`Apache <http://httpd.apache.org>`_ + `mod_wsgi <http://modwsgi.googlecode.com>`_, `Google AppEngine <https://developers.google.com/appengine/>`_, etc).
* Python Paste components (e.g WebOb)

We mostly test on latest releases of `Ubuntu Server <http://www.ubuntu.com/download/server>`_, and Google's `AppEngine <https://developers.google.com/appengine/>`_. 

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

* Include as much detail as you can about your environment (e.g Server OS, Web Server Version, WSGI connector)
* Steps that we can use to replicate the bug
* Share a bit of your application code with us, it goes a long way to replicate issues

Commercial Support
^^^^^^^^^^^^^^^^^^

All commercial endeavors around prestans are managed by Eternity Technologies.

* Help with designing high performance REST apps using prestans
* Extending prestans support for backends, serializers, etc.
* Developer training in Python, prestans, Google Closure.

We also offer custom development services for writing high end Ajax and mobile apps, check out `our website <http://etk.com.au>`_ to see if we can help you create your next Web / Mobile project.

