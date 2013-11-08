============
Installation
============

For a typical server production deployment, we recommend installing prestans via `PyPI <https://pypi.python.org/pypi/prestans>`_::

    $ sudo pip install prestans

this will build and install prestans for your default Python interpreter. To keep up-to-date; use the ``---upgrade`` option::

	$ sudo pip install prestans --upgrade

If you need to run multiple versions of prestans on a server, consider using `Virtualenv <http://www.virtualenv.org/en/latest/>`_.

.. warning:: prestans 2.0 is backwards incompatible. 1.x is still available for download, upgrading to 2.x will break your 1.x application.

Alternatively you can download and build prestans using distutils::

    $ tar -zxvf prestans-2.0.tgz
    $ cd prestans-2.0
    $ sudo python setup.py install

Environments like Google's AppEngine require you to include custom packages as part of your source. Things to consider when distributing prestans with your application:

* Make sure you target a particular release of prestans, distributing our development branch is not recommended (unless of course you require a bleeding edge feature). 
* If you prefer to include prestans as a Git submodule, ensure you use reference one of the ``tags``.
* If your server environment has hard limits on number of files, consider using `zipimport <http://docs.python.org/2/library/zipimport.html>`_.

As a `Git submodule <http://git-scm.com/book/en/Git-Tools-Submodules>`_::

	$ git submodule add https://github.com/prestans/prestans.git prestans

When including prestans manually ensure that your web server is able to locate the files. Read on for tips on configuring Apache and AppEngine for prestans based projects.

Software Requirements
=====================

The server side requires a WSGI compliant environment:

* Python 2.7
* WSGI compliant server environment (`Apache <http://httpd.apache.org>`_ + `mod_wsgi <http://modwsgi.googlecode.com>`_, or `Google AppEngine <https://developers.google.com/appengine/>`_, etc).
* WebOb 1.2.3 (if you're using PyPI this should be installed as a dependency, AppEngine already provides WebOb)
* You may optionally need SQLAlchemy or AppEngine Datastore.

:doc:`client` talks about client side integration with Google Closure projects.

We mostly test on latest releases of `Ubuntu Server <http://www.ubuntu.com/download/server>`_, and Google's `AppEngine <https://developers.google.com/appengine/>`_.

Starting your project
=====================


For deployment under Apache
---------------------------

Directory stucture::

	+-- app
	+-- conf
	+-- static
	+-- ext
	    +-- prestans
	+-- client
	+-- conf

`mod_wsgi <http://code.google.com/p/modwsgi/wiki/ConfigurationDirectives#WSGIPythonPath>`_

For deployment under AppEngine
------------------------------

Python projects under AppEngine use a YAML configuration file called `app.yaml <https://developers.google.com/appengine/docs/python/config/appconfig>`_ to specify versions of Python libraries you project requires. Ensure that you have one of the following included in your `app.yaml <https://developers.google.com/appengine/docs/python/config/appconfig#Python_app_yaml_Configuring_libraries>`_ file.

During development::

    libraries:
    - name: webob
      version: latest

During deployment::

    libraries:
    - name: webob
      version: "1.2.3"


Leaving this directive out loads version 1.1 of WebOb; prestans 2.0 onwards specifically uses WebOb 1.2.3+.

Development Server
==================

* `werkzeug <http://werkzeug.pocoo.org/>`_ - built on top of Pocoo's web server.
* `voluptuous <https://github.com/alecthomas/voluptuous>`_ - Voluptuous, despite the name, is a Python data validation library. It is primarily intended for validating data coming into Python as JSON, YAML, etc.

