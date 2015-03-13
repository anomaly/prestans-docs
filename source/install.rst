============
Installation
============

For a typical server production deployment, we recommend installing Prestans via `PyPI <https://pypi.python.org/pypi/prestans>`_::

    $ sudo pip install prestans

this will build and install Prestans for your default Python interpreter. To keep up-to-date; use the ``---upgrade`` option::

	$ sudo pip install Prestans --upgrade

If you need to run multiple versions of Prestans on a server, consider using `Virtualenv <http://www.virtualenv.org/en/latest/>`_.

.. warning:: Prestans 2.0 is backwards incompatible. 1.x is still available for download, upgrading to 2.x will break your 1.x application.

Alternatively you can download and build Prestans using distutils::

    $ tar -zxvf prestans-2.x.x.tgz
    $ cd prestans-2.x.x
    $ sudo python setup.py install

Environments like Google's AppEngine require you to include custom packages as part of your source. Things to consider when distributing Prestans with your application:

* Make sure you target a particular release of prestans, distributing our development branch is not recommended (unless of course you require a bleeding edge feature). 
* If you prefer to include Prestans as a Git submodule, ensure you use reference one of the ``tags``.
* If your server environment has hard limits on number of files, consider using `zipimport <http://docs.python.org/2/library/zipimport.html>`_.

As a `Git submodule <http://git-scm.com/book/en/Git-Tools-Submodules>`_::

	$ git submodule add https://github.com/anomaly/prestans.git ext/prestans

When including Prestans manually ensure that your web server is able to locate the files. 

Software Requirements
=====================

The server side requires a WSGI compliant environment:

* Python 2.7
* WSGI compliant server environment (`Apache <http://httpd.apache.org>`_ + `mod_wsgi <http://modwsgi.googlecode.com>`_, or `Google AppEngine <https://developers.google.com/appengine/>`_, etc).
* WebOb 1.2.3 (if you're using PyPI this should be installed as a dependency, AppEngine already provides WebOb)
* Your choice of a persistent store might require you to install `SQLAlchemy <http://www.sqlalchemy.org/>`_ or AppEngine `Datastore <https://developers.google.com/appengine/docs/python/datastore/>`_.

:doc:`client` covers our client side Javascript library for Google Closure projects.

We mostly test on latest releases of `Ubuntu Server <http://www.ubuntu.com/download/server>`_, and Google's `AppEngine <https://developers.google.com/appengine/>`_.

Deployment notes
================

The following are environment specific deployment gotchas. Things we've learnt the hard way. We'll ensure to constantly keep updating this section as we find more, we also encourage you to keep a eye out on our mailing lists.

Apache + mod_wsgi
-----------------

Consider the following directory stucture, you might wish to checkout a bleeding edge Prestans into a source controlled directory (in this instance Git)::

	+-- app
	+-- conf
	+-- static
	+-- ext
	    +-- prestans
	+-- client
	+-- conf
        project.pth

mod_wsgi's `WSGIPythonPath <http://code.google.com/p/modwsgi/wiki/ConfigurationDirectives#WSGIPythonPath>`_ directive tells mod_wsgi to add any locations delclared in files with the ``.pth`` extension to the runtime Python path. This allows you to put Python modules in a directory e.g ``ext`` and distribute it with your project.

AppEngine
---------

Python projects under AppEngine use a YAML configuration file called `app.yaml <https://developers.google.com/appengine/docs/python/config/appconfig>`_ to specify versions of Python libraries you project requires. Ensure that you have one of the following included in your app.yaml file.

During development::

    libraries:
    - name: webob
      version: latest

During deployment::

    libraries:
    - name: webob
      version: "1.2.3"


Leaving this directive out loads version 1.1 of WebOb; Prestans 2.0 onwards specifically uses WebOb 1.2.3+.


Unit testing
============

prestans ships a `unit testing suite <https://docs.python.org/2/library/unittest.html>`_, due to the nature of the project only the following can be reliably unit tested:

* Prestans Data Type validation

to run the testsuite checkout Prestans via Git and use the Python unitest framework:

.. code-block:: bash

    python -m unitest prestans.testsuite


