=================
Securing your API
=================

Each project has very different requirements for authentication and more importantly each developer likes to implement each scenario differently. The Python Web world is a world of micro frameworks that work toegether in harmony. prestans does not implement any authentication mechanisms, in turn it implements a set of patterns called ``Providers`` (refer to our :doc:`concepts` chapter) that assist in making prestans application respect your application's chosen authentication method. 

What this means is prestans provides you the opportunity to tell it what your application considers, authenticated and authorized. Your prestans REST handlers use a set of predefine decorators to communicate with your prestans application's authentication provider to secure REST end points.

Security typically has two parts to the problem, Authentication to see if the user is allowed in the system at all, and Authorization to see if the user is allowed to access a particular resource. If you are checking for Authority, prestans assumes that the user is required to be authenticated.

Security is defined per HTTP method of a handler (e.g a User can read a list of resources, but is not allowed to update them), prestans provides a set of decorators that your REST handler methods use to express their authentication/authorization requirements.

Fitting into your environment
=============================

An API end point should respond if the user is unauthenticated, obviously with a message to tell them they are unauthenticated. API's are client agnostic, so It's nearly "*never*"" the API end point's responsibility to send the user to a login page. If a uesr is accessing a resource they are not meant to be, prestans will send a properly formed message as the response.

``prestans.auth`` provides a stub for the ``AuthContextProvider``, this class is never meant to be used directly, your application is expected to provide a ``class`` that extends from ``AuthContextProvider``.

``AuthContextProvider`` defines the following method stubs

.. code-block:: python

    class AuthContextProvider:
        
        def is_authenticated_user(self, handler_reference):
            raise Exception("Direct use of AuthContextProvider not allowed")

        def get_current_user(self):
            raise Exception("Direct use of AuthContextProvider not allowed")

        def current_user_has_role(self, role_name):
            raise Exception("Direct use of AuthContextProvider not allowed")        
            
Let's discuss these in order of relevance:

* ``is_authenticated_user`` must return ``True`` or ``False`` to indicate if a user is current logged in, the function is additionally provided a reference to the hander. Your application has the opportunity to use any supporting libraries to determine if the user is logged in and return a response.

* ``get_current_user`` should return a reference to the ``user`` object for your application. This can be a persistent object, or user identifier, whatever your application would find most useful when persistenting data.

* ``current_user_has_role`` is provided a set of rolenames that the handle is allowed to use, role_name can be a refernce to a list of strings, constants whatever your app deems relevant. This method will only run after prestans has checked that the user is authenticated. Obviously you can use ``self.get_current_user`` to get a reference tot he currently logged in user.

Writing your own provider
-------------------------

Writing your own ``AuthContextProvider`` comprises of overriding the three methods discussed in the previous section. The following example demonstates the use of `Beaker Sessions <http://beaker.groovie.org>`_ to validate if the user is logged in. Notice that that most of the code is referencee from another package that provides authentication information to pages rendered by handlers.

The ``__init__`` method is not used by the parent class, so if you need to pass extra references to objects that you need to use to perform the authentication here's the place to do it.

.. code-block:: python

    class MyAuthContextProvider(prestans.auth.AuthContextProvider):
        
        ## @brief custom constructor that takes in a reference to the beaker environment var
        #
        def __init__(self, environ):
            self._environ = environ
        
        ## @brief checks to see if a Beaker session is set or not
        #
        # Beaker session reference is passed into the constructor and made available as
        # an instance variable to the auth context provider
        #
        def is_authenticated_user(self):
            return self._environ and self._environ.get(myapp.auth.SESSION_KEY)
            
        ## @brief returns a user object from myapp.models
        def get_current_user(self):
            remote_user = self._environ.get(myapp.auth.SESSION_KEY)
            return myapp.auth.get_userprofile_by_username(remote_user)


Working with Google AppEngine
-----------------------------

prestans ships with an inbuilt provider for Google AppEngine. AppEngine is a WSGI environment and has a very fixed authentication lifecycle encapsulated by ``prestans.ext.appengine.AppEngineAuthContextProvider``. The AppEngine AuthContextProvider implements support for OAuth and Google account authentication.

Obviously this does not implement the ``current_user_has_role``. If you wish to support role based authorization you must extend this class and implement this function.

Attaching AuthContextProvider to Handlers
=========================================

Like all things prestans, attaching a auth context provider to a handler is as simple as assigning an instance of your ``AuthContextProvider`` to your ``RESTRequestHandler``'s auth_context property::

    class MyHandler(prestans.handlers.RESTRequestHandler):

        auth_context = myapp.auth.MyAuthContextProvider()
        
This tells your handler which ``AuthContextProvider`` to use. Remember that authentication configuration is per HTTP method supported by your request handler:

* If your handler method just wants to ensure that a user is logged in, all you need to do is decorate your HTTP method with ``@prestans.auth.login_required``.

* If your handler method wants to test final grained roles use the ``@prestans.auth.role_required`` decorator. This implies that a user is already logged in.

The following example allows any logged in user to get resources, users with role authors to create and update resources, but only users with role admin to delete resources.

.. code-block:: python

    class MyRESTHandler(prestans.handlers.RESTRequestHandler):

        auth_context = myapp.auth.MyAuthContextProvider()

        @prestans.auth.login_required
        def get(self):
            .... do what you need to here

        @prestans.auth.role_required(role_name=['authors'])
        def post(self):
            .... do what you need to here

        @prestans.auth.role_required(role_name=['authors'])
        def put(self):
            .... do what you need to here

        @prestans.auth.role_required(role_name=['admin'])
        def delete(self):
            .... do what you need to here

