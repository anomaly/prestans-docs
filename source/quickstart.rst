===============
Getting Started
===============

prestans is a WSGI compliant REST server framework, best suited for use with applications that nearly their entire interface using JavaScript (using frameworks like `Google Closure <https://developers.google.com/closure/>`_) or a bespoke Mobile client. Although prestans is a standalone framework, it provides hooks (called Providers) to integrate with your application's authentication, caching and other such core services.

We have battle tested prestans under Apache (using mod_wsgi) and Google's AppEngine platform.

Code samples used throughout our documentation is available as a `Google AppEngine project <https://code.google.com/p/prestans-demo/>`_, we highly recommend you grab a copy so you can see how it all fits in.

.. note:: You will require a copy of `Google's AppEngine Python SDK <https://developers.google.com/appengine/downloads>`_ (v1.7.0+) to run the sample project.

A downloadable release is available for prestans demo, but we are consistentantly working on it, so you might want to check out the Subversion repository.

Our philosophy is **"take as much or as little of the project as you like"**, prestans was designed ground up to sit nicely along side other Python frameworks. Needless to say that a dynamic language such as Python lends itself extremely well to writing frameworks such as prestans, and highly scaleable Web applications.

And incase you are still wondering prestans is a latin word meaning *"excellent, distinguished, imminent".*

prestans is distributed under the terms and conditions of the New BSD license and is hosted on Google Code.

Features
========

* Validation or incoming and outgoing using strongly defined Models
* Pluggable architecture allowing prestans to plug into any authentication, caching and serialization requirements.
* A custom URL dispatcher that allows you to re-use handlers for multiple output formats.
* Data Adapters, that allows you to translate persistent objects into REST resources, with a single line of code.
* Validation of URL parameters using strong defined Parameter Sets.
* Dynamically filtering response fields when writing responses.

We also maintain a set of tools that leverages prestans's ``Model`` definition schema to generate boiler plate client side parsing of REST resources.

Sample App Scenario
===================

Due to our obsession with music, we thought it be fitting to build a music catalgoue as our demonstration application. The application models Artists, Bands, Albums and Tracks to demonstrate the features and techniques to make REST based Web applications.

Sample data features legendary artists and bands like `Pink Floyd <http://en.wikipedia.org/wiki/Pink_Floyd>`_, `Eric Clapton <http://en.wikipedia.org/wiki/Eric_Clapton>`_ and `Metallica <http://en.wikipedia.org/wiki/Metallica>`_ purely due to the developers bias in music.

It's complete with a Google Closure based user interface, which shows off the set of handy automation tools that prestans ships to speed up client side development.

.. note:: Subversion path for our demo app, https://prestans-demo.googlecode.com/svn/trunk/

The demo app ships with it's own copy of prestans, once you've obtained a copy of the demo app, and assuming you have Google's AppEngine Python SDK setup, just run the following command::

    $ cd prestans-demo/app
    $ dev_appsever.py .

Navigate to http://localhost:8080/ and voilà we have an app!

Installation
============

There are two ways you can install prestans, one using Python's distutils and the other is by simple including it with your project's source. This is pariticularly handy if you are using environments like Google's AppEngine.

Making prestans available system wide, download a copy of the latest release, unpack the tarball and run setup.py::

    $ cd prestans-1.0
    $ python setup.py install

Ensure that the version of Python you are installing prestans for is the version used by your Web server.

Things to consider when distributing prestans with your application:

* Make sure download target a particular release of prestans, it is not recommended to use the bleeding edge.
* If you prefer reference prestans as a Subversion external, ensure you use reference one of the ``tags``, it is not recommended to reference ``trunk``

