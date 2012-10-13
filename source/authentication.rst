=================
Securing your API
=================

In our experience each project has very different requirements for authentication and more importantly each developer likes to implement each scenario differently. The Python Web world is a world of micro frameworks that work toegether in harmony. prestans does not implement any authentication mechanisms, in turn it implements a set of patterns called ``Providers`` (refer to our :doc:`concepts` chapter) that assist in making prestans application respect your application's chosen authentication method. 

What this means is prestans provides you the opportunity to tell it what your application considers, authenticated and authorized. Your prestans REST handlers use a set of predefine decorators to communicate with your prestans application's authentication provider to secure REST end points.

Security typically has two parts to the problem, Authentication to see if the user is allowed in the system at all, and Authorization to see if the user is allowed to access a particular resource. If you are checking for Authority, prestans assumes that the user is required to be authenticated.

Security is defined per HTTP method of a handler (e.g a User can read a list of resources, but is not allowed to update them), prestans provides a set of decorators that your REST handler methods use to express their authentication/authorization requirements.

Authentication
==============



Authorisation
=============

To use Authorization you must ensure that a user is authenticated.

Fitting into your environment
=============================


Working with Google AppEngine
-----------------------------

Writing your own Providers
==========================