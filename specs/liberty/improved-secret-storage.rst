..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Improved secret storage utilizing Barbican
==========================================

https://blueprints.launchpad.net/sahara/+spec/improved-secret-storage

There are several secrets (for example passwords) that Sahara uses with
respect to deployed frameworks which are currently stored in its
database. This blueprint proposes the creation of a Barbican integration
module that will allow Sahara to offload secret storage to that service.


Problem description
===================

There are several situations under which Sahara stores usernames and
passwords to the database. The storage of these credentials represents
a security risk for any installation that exposes routes to the
controller's database. To reduce the risk of a breach in user
credentials, Sahara should move towards using an external key manager
to control the storage of user passwords.


Proposed change
===============

This specification proposes the creation of a new module in sahara that
will provide access to an external key manager, initially implemented
with Barbican. Usage of this new module will be disabled by default with
an option to enable in the configuration file.

The secrets module will be an abstracted wrapper around the
Barbican client. The module will provide access to create, retrieve,
and destroy secrets based on unique identifiers provided by the
key manager. It will not store information in the Sahara database.
The secrets module will use Sahara's admin credentials to authenticate
with the key manager.

The rationale for an abstraction around the Barbican client is to
allow for changing the key manager without disrupting access for
dependent modules in Sahara.

The intended use of this module will be to convert any secrets that are
stored within the Sahara database to instead be stored as unique
identifiers. These identifiers will then be used to retrieve the secrets.
The storage of secrets will be based on the notion of storing any data
that can be represented as a string, providing it does not violate the
storage requirements of the external key manager.

Example secrets module usage::

    # store a secret from a password
    from sahara.utils import secrets
    new_secret_id = secrets.store('password_text_here')

    # retrieve a secret
    secret_cleartext = secrets.get(new_secret_id)

    # revoke a secret
    secrets.destroy(new_secret_id)

This solution will offload the secrets in such a manner that an
attacker would need to penetrate the database and learn the Sahara
admin credentials to gain access to the stored passwords. In essence
we are adding one more block in the path of a would-be attacker.

Alternatives
------------

One possible alternative to using an external key manager would be
for Sahara to encrypt passwords and store them in Swift. This would
satisfy the goal of removing passwords from the Sahara database
while providing a level of security from credential theft.

The downside to this methodology is that it still places Sahara in
the position of arbitrating security transactions. Namely, the use
of cryptography in the creation of the stored password data.

Data model impact
-----------------

None.

REST API impact
---------------

None.

Other end user impact
---------------------

None.

Deployer impact
---------------

A new configuration option will be provided to enable the use of
an external key manager. Deployers will need to be aware of, and
install a key manager in their stacks.

Developer impact
----------------

None.

Sahara-image-elements impact
----------------------------

None.

Sahara-dashboard / Horizon impact
---------------------------------

None.


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  mimccune (Michael McCune)

Work Items
----------

* create Barbican client utility module
* create secrets module
* add tests for secrets module
* create documentation for external key manager usage


Dependencies
============

None.


Testing
=======

Unit tests will be created to exercise the secrets module and ensure that
that stored identifiers can be referenced properly.

Ideally integration tests will be created to ensure the proper storage
and retrieval of secrets. The addition of these tests represents a
larger change to the testing infrastructure as Barbican will need to be
added. Depending on the impact of changing the testing deployment these
might best be addressed in a separate change.


Documentation Impact
====================

A new section in the advanced configuration guide will be created to
describe the usage of this new feature.

Additionally this feature should be described in the OpenStack
Security Guide. This will require a separate change request to the
documentation project.


References
==========

* Barbican documentation http://docs.openstack.org/developer/barbican/
* Barbican wiki https://github.com/cloudkeep/barbican/wiki
