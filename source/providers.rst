=========
Providers
=========

Prestans is a micro-framework designed to streamline building REST APIs. Providers are Prestans' bridge to other infrastructure level services that are specific to each application but are required for REST APIs to function.

Authentication
==============

Provided by the ``prestans.provider.auth`` package and is designed to plug into an existing authentication system. Prestans does not provide an authentication layer because it's understood that every applicaiton and environment has it's own requirements, and this outside the realm of Prestans.

The package provides two authentication decorators (see :pep::`318`):

* ``login_required`` 
* ``role_required``


AppEngine extension
^^^^^^^^^^^^^^^^^^^

