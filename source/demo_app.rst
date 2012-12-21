=============================
Demo Application (incomplete)
=============================

Due to our obsession with music, we thought it be fitting to build a music catalgoue as our demonstration application. The application models Artists, Bands, Albums and Tracks to demonstrate the features and techniques to make REST based Web applications.

Sample data features legendary artists and bands like `Pink Floyd <http://en.wikipedia.org/wiki/Pink_Floyd>`_, `Eric Clapton <http://en.wikipedia.org/wiki/Eric_Clapton>`_ and `Metallica <http://en.wikipedia.org/wiki/Metallica>`_ purely due to the developers bias in music.

It's complete with a Google Closure based user interface, which shows off the set of handy automation tools that prestans ships to speed up client side development.

.. note:: Subversion path for our demo app, https://prestans-demo.googlecode.com/svn/trunk/

The demo app ships with it's own copy of prestans, once you've obtained a copy of the demo app, and assuming you have Google's AppEngine Python SDK setup, just run the following command::

    $ cd prestans-demo/app
    $ dev_appsever.py .

Navigate to http://localhost:8080/ and voilà we have an app!