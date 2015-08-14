..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

================================================
Updating authentication to use keystone sessions
================================================

https://blueprints.launchpad.net/sahara/+spec/keystone-sessions

Sahara currently uses per access authentication when creating OpenStack
clients. This style of authentication requires keystone connections on every
client creation. The keystone project has created a mechanism to streamline
and improve this process in the form of Session objects. These objects
encapsulate mechanisms for updating authentication tokens, caching of
connections, and a single point for security improvements. Sahara should
migrate its OpenStack client objects to use session objects for all clients.


Problem description
===================

For all OpenStack client instances, sahara uses authentication on a
per-client creation basis. For each client object that is requested, a set of
credentials are acquired from the context, or the configuration file in the
case of admin accounts, which are used to initialize the client object.
During this initialization a request is made to the Identity service to
determine the user's privileges with respect to the new client.

Sahara must be aware of any changes to the authentication methods for each
client as well as any potential security vulnerabilities resulting from the
usage of those methods.

This method of authentication does not allow sahara to share any information
between clients, aside from the raw credentials. In turn, this introduces
brittleness to the sahara/client interface as each authentication
relationship must be maintained separately or, worse yet, with a partial
shared model.

Having separate interfaces for each client also makes applying security
updates more difficult as each client instance must be visited, researched,
and ultimately fixed according the specific details for that client.

Although this methodology has served sahara well thus far, the keystone
project has introduced new layers of abstraction to aid in sharing common
authentication between clients. This shared methodology, the keystoneclient
Session object, provides a unified point of authentication for all clients.
It serves as a single point to contain security updates, on-demand
authentication token updating, common authentication methods, and
standardized service discovery.


Proposed change
===============

Sahara should standardize its client authentication code by utilizing
keystoneclient Session objects. This change will entail creating a new
module, modifying the OpenStack client utility functions, and adding
an authentication plugin object to the context.

A new module, ``sahara.service.sessions``, will be created to contain utility
functions and classes to aid in the creation and storage of session objects.
This module will also contain a global singleton for the sessions cache.

``sahara.service.sesssions`` will provide a class named ``SessionCache`` as
well as a function to gain the global singleton instance of that class. The
``SessionCache`` will contain cached session objects that can be reused in
the creation of individual OpenStack clients. It will also contain functions
for generating the session objects required by specific clients. Some clients
may require unique versions to be cached, for example if a client requires a
specific certificate file then it may have a unique session. For all other
clients that do not require a unique session, a common session will be used.

Authentication for session objects will be provided by one of a few methods
depending on the type of session needed. For user based sessions,
authentication will be obtained from the keystonemiddleware authentication
plugin that is generated with each request. For admin based sessions, the
credentials found in the sahara configuration file will be used to generate
the authentication plugin. Trust based authentication will be handled by
generating an authentication plugin based on the information available in
each case, either a token for a user or a password for an admin or proxy
user.

The ``sahara.context.Context`` object will be changed to incorporate an
authentication plugin object. When created through REST calls the
authentication plugin will be obtained from the keystonemiddleware. When
copying a context, the authentication plugin will be copied as well. For
other cases the authentication plugin may be set programmatically, for
example if an admin authentication plugin is required it can be generated
from values in the configuration file, or if a trust based authentication
is required it can be generated.

The individual OpenStack client utility modules will be changed to use
session and authentication plugin objects for their creation. The
sessions will be obtained from the global singleton and the authentication
plugin objects can be obtained from the context, or created in circumstances
that require more specific authentication, for example when using the
admin user or trust based authentication.

The clients for heat and swift do not yet enable session based
authentication. These clients should be monitored for addition of this
feature and migrated when available.

Alternatives
------------

An alternative to this approach would be to create our own methodology for
storing common authentication credentials, but this would be an exercise in
futility as we would merely be replicating the work of keystoneclient.

Data model impact
-----------------

None

REST API impact
---------------

None

Other end user impact
---------------------

None

Deployer impact
---------------

None

Developer impact
----------------

None

Sahara-image-elements impact
----------------------------

None

Sahara-dashboard / Horizon impact
---------------------------------

None

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  Michael McCune (elmiko)

Other contributors:
  Andrey Pavlov (apavlov)

Work Items
----------

* create sahara.service.sessions
* modify context to accept authentication plugin
* modify sahara.utils.openstack clients to use sessions

  * cinder
  * keystone
  * neutron
  * nova

* modify admin authentications to use plugin objects
* modify trust authentications to use plugin objects
* create tests for session cache
* create developer documentation for client usage


Dependencies
============

None


Testing
=======

The tests created for this feature will be unit based, to exercise the code
paths and logic points. Functional testing should not be necessary as these
authentication methods will be exercised in the course of the standard
functional testing.


Documentation Impact
====================

This change will only create documentation within the sahara project.
Currently there exists no documentation about client usage within the sahara
codebase. This change will add a small section describing how to instantiate
clients using the ``sahara.utils.openstack`` package, with a note about
common session authentication.


References
==========

`Keystoneclient documentation about using Sessions <http://docs.openstack.org/developer/python-keystoneclient/using-sessions.html>`_

`How to Use Keystoneclient Sessions (article by Jamie Lennox) <http://www.jamielennox.net/blog/2014/09/15/how-to-use-keystoneclient-sessions/>`_
