..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===========================================
Improved secret storage utilizing castellan
===========================================

https://blueprints.launchpad.net/sahara/+spec/improved-secret-storage

There are several secrets (for example passwords) that sahara uses with
respect to deployed frameworks which are currently stored in its database.
This blueprint proposes the usage of the castellan package key manager
interface for offloading secret storage to the OpenStack Key management
service.


Problem description
===================

There are several situations under which sahara stores usernames and
passwords to its database. The storage of these credentials represents
a security risk for any installation that exposes routes to the
controller's database. To reduce the risk of a breach in user
credentials, sahara should move towards using an external key manager
to control the storage of user passwords.


Proposed change
===============

This specification proposes the integration of the castellan package into
sahara. Castellan is a package that provides a single point of entry to
the OpenStack Key management service. It also provides a pluggable key
manager interface allowing for differing implementations of that service,
including implementations that use hardware security modules (HSMs) and
devices which support the key management interoperability protocol (KMIP).

Using the pluggable interface, a sahara specific key manager will be
implemented that will continue to allow storage of secrets in the
database. This plugin will be the default key manager to maintain
backward compatibility, furthermore it will not require any database
modification or migration.

For users wishing to take advantage of an external key manager,
documentation will be provided on how to enable the barbican key
manager plugin for castellan. Enabling the barbican plugin requires
a few modifications to the sahara configuration file. In this manner,
users will be able to customize the usage of the external key manager
to their deployments.

Example default configuration::

    [key_manager]
    api_class = sahara.utils.key_manager.sahara_key_manager.SaharaKeyManager

Example barbican configuration::

    [key_manager]
    api_class = castellan.key_manager.barbican_key_manager.BarbicanKeyManager


To accomodate the specific needs of sahara, a new class will be created
for interacting with castellan; ``SaharaKeyManager``. This class will
be based on the abstract base class ``KeyManager`` defined in the
castellan package.

The ``SaharaKeyManager`` class will implement a thin layer around the storage
of secrets without an external key manager. This class will allow sahara
to continue operation as it exists for the Kilo release and thus maintain
backward compatibility. This class will be the default plugin implementation
to castellan.

Example usage::

    from castellan import key_manager as km
    from castellan.key_manager.objects import passphrase

    keymanager = km.API()

    # create secret
    new_secret = passphrase.Passphrase('password_text_here')

    # store secret
    new_secret_id = keymanager.store_key(context, new_secret)

    # retrieve secret
    retrieved_secret = keymanager.get_key(context, new_secret_id)
    secret_cleartext = retrieved_secret.get_encoded()

    # revoke secret
    keymanager.delete_key(context, new_secret_id)


This solution will provide the capability, through the barbican plugin, to
offload the secrets in such a manner that an attacker would need to
penetrate the database and learn the sahara admin credentials to gain
access to the stored passwords. In essence we are adding one more block
in the path of a would-be attacker.

This specification focuses on passwords that are currently stored in the
sahara database. The following is a list of the passwords that will be moved
to the key manager for this specification:

* Swift passwords entered from UI for data sources
* Swift passwords entered from UI for job binaries
* Proxy user passwords for data sources
* Proxy user passwords for job binaries
* Hive MySQL passwords for vanilla 1.2.1 plugin
* Hive database passwords for CDH 5 plugin
* Hive database passwords for CDH 5.3.0 plugin
* Hive database passwords for CDH 5.4.0 plugin
* Sentry database passwords for CDH 5.3.0 plugin
* Sentry database passwords for CDH 5.4.0 plugin
* Cloudera Manager passwords for CDH 5 plugin
* Cloudera Manager passwords for CDH 5.3.0 plugin
* Cloudera Manager passwords for CDH 5.4.0 plugin

Alternatives
------------

One possible alternative to using an external key manager would be
for sahara to encrypt passwords and store them in swift. This would
satisfy the goal of removing passwords from the sahara database
while providing a level of security from credential theft.

The downside to this methodology is that it places sahara in the position
of arbitrating security transactions. Namely, the use of cryptography in
the creation and retrieval of the stored password data.

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

A new configuration option will be provided by the castellan package to
set the key manager implementation. This will be the SaharaKeyManager by
default. Deployers wishing to use barbican might need to set a few more
options depending on their installation. These options will be discussed
in the documentation.

Use of an external key manager will depend on having barbican installed
in the stack where it will be used.

Developer impact
----------------

Developers adding new stored passwords to sahara should always be using
the key manager interface.

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

Other contributors:
  None

Work Items
----------

* create SaharaKeyManager class
* add tests for new class
* add tests for secret storage
* create documentation for external key manager usage
* migrate passwords to key manager


Dependencies
============

Castellan package, available through pypi. Currently this version (0.1.0)
does not have a barbican implementation, but it is under review[1].


Testing
=======

Unit tests will be created to exercise the SaharaKeyManager class. There
will also be unit tests for the integrated implementation.

Ideally, functional integration tests will be created to ensure the
proper storage and retrieval of secrets. The addition of these tests
represents a larger change to the testing infrastructure as barbican will
need to be added. Depending on the impact of changing the testing
deployment these might best be addressed in a separate change.


Documentation Impact
====================

A new section in the advanced configuration guide will be created to
describe the usage of this new feature.

Additionally this feature should be described in the OpenStack
Security Guide. This will require a separate change request to the
documentation project.


References
==========

[1]: https://review.openstack.org/#/c/171918

castellan repository https://github.com/openstack/castellan

*note, the castellan documentation is still a work in progress*

barbican documentation http://docs.openstack.org/developer/barbican/

barbican wiki https://github.com/cloudkeep/barbican/wiki
