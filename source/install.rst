============
Installation
============

We recommend installing prestans via `PyPI <http://pypi.python.org/pypi>`_::

    $ sudo pip install prestans

this will build and install prestans for your default Python interpreter. To keep up-to-date; use the ``---upgrade`` option::

	$ sudo pip install prestans --upgrade

Alternatively you can download and build prestans using distutils::

    $ tar -zxvf prestans-2.0.tgz
    $ cd prestans-2.0
    $ sudo python setup.py install

Environments like Google's AppEngine require you to include custom packages as part of your source. Things to consider when distributing prestans with your application:

* Make sure you target a particular release of prestans, distributing our development branch is not recommended (unless of course you require a bleeding edge feature). 
* If you prefer to include prestans as a Git submodule, ensure you use reference one of the ``tags``.
* If your server environment has hard limits on number of files, consider using `zipimport <http://docs.python.org/2/library/zipimport.html>`_.

As a `Git submodule <http://git-scm.com/book/en/Git-Tools-Submodules>`_::

	$ git submodule add git://github.com/prestans/prestans.git prestans

Make sure you read on about configuring your Web server environment.

Software Requirements
=====================

The server side requires a WSGI compliant environment:

* Python 2.7; should work under *2.6* also
* WSGI compliant server environment (`Apache <http://httpd.apache.org>`_ + `mod_wsgi <http://modwsgi.googlecode.com>`_, or `Google AppEngine <https://developers.google.com/appengine/>`_, etc).
* WebOb 1.2.3 (should be installed as a dependency, AppEngine already provided WebOb)
* You may optionally need SQLAlchemy or AppEngine.

Client side code is written for Google Closure.

We mostly test on latest releases of `Ubuntu Server <http://www.ubuntu.com/download/server>`_, and Google's `AppEngine <https://developers.google.com/appengine/>`_.

Starting your project
=====================

For deployment under Apache
---------------------------

Directory stucture::

	+-- app
	+-- static
	+-- ext
	    +-- prestans

For deployment under AppEngine
------------------------------


Development Server
==================

prestans ships with a development server built on top of Pooco's `werkzeug <http://werkzeug.pocoo.org/>`_. 


Apache Configuration tips
=========================
