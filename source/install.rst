============
Installation
============

We recommend installing prestans via `PyPI <http://pypi.python.org/pypi>`_::

    $ pip install prestans

this will build and install prestans for your default Python interpreter.

Alternatively you can download and build prestans using distutils::

    $ tar -zxvf prestans-2.0.tgz
    $ cd prestans-2.0
    $ python setup.py install

Environments like Google's AppEngine require you to include custom packages as part of your source. Things to consider when distributing prestans with your application:

* Make sure you target a particular release of prestans, distributing our development branch is not recommended. 
* If you prefer reference prestans as a Subversion external, ensure you use reference one of the ``tags``, it is not recommended to reference ``trunk``
* If your server environment has hard limits on number of files, consider using `zipimport <http://docs.python.org/2/library/zipimport.html>`_.

Software Requirements
---------------------

The server side requires a WSGI compliant environment:

* Python 2.6+, *2.7 recommended*
* WSGI compliant server environment (`Apache <http://httpd.apache.org>`_ + `mod_wsgi <http://modwsgi.googlecode.com>`_, or `Google AppEngine <https://developers.google.com/appengine/>`_, etc).
* Python Paste components (e.g WebOb)

Client side code is written for Google Closure.

We mostly test on latest releases of `Ubuntu Server <http://www.ubuntu.com/download/server>`_, and Google's `AppEngine <https://developers.google.com/appengine/>`_. 
