=====================================================
[EDP] Using trust delegation for Swift authentication
=====================================================


https://blueprints.launchpad.net/sahara/+spec/edp-swift-trust-authentication


Sahara currently stores and distributes credentials for access to Swift
objects. These credentials are username/password pairs that are stored in
Sahara's database. This blueprint describes a method for using Keystone’s
trust delegation mechanism in conjuction with temporary proxy users to remove
the storage of user credentials from Sahara's purview.

Problem description
===================

Sahara allows access to job binaries and data sources stored in Swift
containers by storing a user's credentials to those containers. The
credentials are stored in Sahara’s database and distributed to cluster
instances as part of a job’s workflow, or used by Sahara to access job
binaries. The storage of user credentials in Sahara's database represents a
security risk that can be avoided.

Proposed change
===============

A solution to using credentials for access to Swift objects is to generate a
Keystone trust between the user with access to those objects and a Sahara
proxy user. The trust would be established based on the user’s membership in
the project that contains the Swift objects. Using this trust the Sahara proxy
user could generate authentication tokens to access the Swift objects. When
access is no longer needed the trust can be revoked, and the proxy user
removed thus invalidating the tokens.

A proxy user will be created per job execution, and belong to a roleless
domain created by the stack administrator for the express purpose of
containing these new users. This domain will allow Sahara to create users as
needed for the purpose of delegating trust to access Swift objects.

Sahara will generate usernames and passwords for the proxy users when they are
created. These credentials will be used in conjuction with the trust to allow
Swift accress from the cluster instances.

General breakdown of the process:

1. On start Sahara confirms the existence of the proxy domain. If no proxy
   domain is found and the user has configured Sahara to use one, then Sahara
   will log an error and resume in backward compatibility mode.

2. When a new job that involves Swift objects is executed a trust is created
   between the context user and a newly created proxy user. The proxy user's
   name and password are created by Sahara and stored temporarily.

3. When an instance needs access to a Swift object it uses the proxy user's
   credentials and the trust identifier to create the necessary authentication
   token.

4. When the job has ended, the proxy user and the trust will be removed from
   Keystone.

Detailed breakdown:

Step 1.

On start Sahara will confirm the existence of the proxy domain specified by
the administrator in the configuration file. If no domain can be found and the
user has configured Sahara to use one, then it will log an error and resume
in backward compatibility mode. The domain will be used to store proxy users
that will be created during job execution. It should explicitly have an SQL
identity backend, or another backend that will allow Sahara to create
users.(SQL is the Keystone default)

If the user has configured Sahara not to use a proxy domain then it will
fall back to using the backward compatible style of Swift authentication.
Requiring usernames and passwords for Swift access.

Step 2.

Whenever a new job execution is issued through Sahara a new proxy user will
be created in the proxy domain. This user will have a name generated based
on the job execution id, and a password generated randomly.

After creating the proxy user, Sahara will delegate a trust from the current
context user to the proxy user. This trust will grant a "Member" role by
default, but will allow configuration, and impersonate the context user. As
Sahara will not know the length of the job execution, the trust will be
generated with no expiration time and unlimited reuse.

Step 3.

During job execution, when an instance needs to access a Swift object, it
will use the proxy user's name and password in conjuction with the trust
identifier to create an authentication token. This token will then be used
to access the Swift object store.

Step 4.

After the job execution has finished, successfully or otherwise, the proxy
user and trust will be removed from Keystone.

A periodic task will check the proxy domain user list to ensure that none have
become stuck or abandoned after a job execution has been completed.

Alternatives
------------

Three alternatives have been discussed regarding this issue; using Swift’s
TempURL mechanism, encrypting the Swift credentials, and distributing tokens
from Sahara to the cluster instances.

Swift implements a feature named TempURL which allows the generation of
temporary URLs to allow public access to Swift objects. Using this feature
Sahara could create TempURLs for each Swift object that requires access and
then distribute these URLs to the cluster instances in the job workflow.

Although TempURLs would be low impact in terms of the work required to
implement they have a few major drawbacks for Sahara’s use case. When
creating a TempURL an expiration date must be associated with the URL. As
job lengths in Sahara cannot be predictively determined this would mean
creating indefinite expiration dates for the TempURLs. A solution to this
would be deleting the Swift object or changing the authentication identifier
associated with the creation of the TempURL. Both of these options present
implications that run beyond the boundaries of Sahara. In addition the
TempURLs would need to be passed to the instances as ciphertext to avoid
potential security breaches.

Another methodology discussed involves encrypting the Swift credentials and
allowing the cluster instances to decrypt them when access is required. Using
this method would involve Sahara generating a two part public/private key that
would be used to encrypt all credentials. The decryption, or private, part of
the key would be distributed to all cluster nodes. Upon job creation the Swift
credentials associated with the job would be encrypted and stored. The
encrypted credentials would be distributed to the cluster instances in the job
workflow. When access is needed to a Swift object, an instance would decrypt
the credentials using the locally stored key.

Encrypting the credentials poses a lower amount of change over using Keystone
trusts but perpetuates the current ideology of Sahara storing credentials for
Swift objects. In addition a new layer of security management becomes involved
in the form of Sahara needing to generate and store keys for use with the
credentials. This complexity adds another layer of management that could
instead be relegated to a more appropriate OpenStack service(i.e. Keystone).

Finally, another possibilty to achieve the removal of credentials from Sahara
would be for the main controller to distribute preauthorized tokens to the
cluster instances. These tokens would be used by the instances to validate
their Swift access. There are a few implementation problems with this approach
that have caused it to be abandoned by a previous version of this blueprint.

Keystone tokens have a default lifespan of one hour, this value can be
adjusted by the stack administrator. This implies that tokens must be updated
to the instances at least once an hour, possibly more frequently. The updating
process proves to add hindrances that are difficult to overcome. Update
frequency is one, but resource contention on the instances is another. A
further examination of this method shows that the updates to the Hadoop Swift
filesystem component will create a disonant design surrouding the injestion
of configuration data. In sum this methodology proves to be more fragile
than is acceptable.

Data model impact
-----------------

The job execution model currently stores username and password information in
a field that is a dictionary. There will be no changes to the model, but the
trust identifier will need to be stored in addition to the username and
password.

Once the credentials have been passed to the cluster the only values that
need be stored are the user id and trust identifier. These would need to be
used to destroy the trust and user after job execution. The proxy user's
password is only needed during the creation of the job execution and will
be distributed to the instances, but long term storage is not necessary.

REST API impact
---------------

The proxy usernames and trust identifiers should be sanitized from the
job execution output.

Other end user impact
---------------------

Users will no longer need to enter credentials when adding Swift data sources
to their jobs.

The user’s OpenStack credentials will need to have sufficient privileges to
access the Swift objects they add.

From the python-saharaclient, developers will no longer need to enter
credential_user or credential_pass when making a requests to create data
sources.

Keystone deployments that use LDAP backed domains will need to be configured
as recommended by the Keystone group, using domain based configurations. This
ensures that new domains created will be backed by SQL.

Deployer impact
---------------

A deployer will need to be aware of the Keystone configuration with respect
to the default identity backend. They will also need to create the proxy
domain and provide the Sahara service user with enough access to create new
users in that domain.

Developer impact
----------------

Developers will no longer need to pass credentials when creating data sources.

Sahara-image-elements impact
----------------------------

None

Sahara-dashboard / Horizon impact
---------------------------------

For backward compatibility the username and password fields should be left in
the Swift data source forms and views, but they should allow the user to
enter blank data.

Implementation
==============

Assignee(s)
-----------

Primary assignee:

* Michael McCune

Other contributors:

* Trevor McKay


Work Items
----------

* Domain detection and configuration option
* Proxy user creation/destruction
* Trust acquisition/revocation
* Workflow update
* Swift file system component update to use trust identifier
* Periodic proxy user removal task
* Documentation
* Tests

Dependencies
============

This feature will require the usage of Keystone v3 with the OS-TRUST mechanism
enabled.

The Horizon forms for Swift data sources will need to allow blank entries
for username and password.

The Hadoop Swift file system component will need to be updated as well.

Testing
=======

The current tests for Swift based objects will need to be modified to remove
the usage of username/password credentials. Otherwise these tests should prove
that the trust method is working properly.

Tests for situations where a user’s Keystone access do not permit permission
for the Swift objects they are adding should be implemented.

The Swift file system component will need it's tests modified to use
trust identifiers for scoping of the authentication tokens.

Documentation Impact
====================

The documentation for usage of Swift storage will need to have references to
the object credentials removed. Additionally there should be documentation
added about the impact of a user not having access to the Swift sources.

The proxy domain usage should be documented to give stack administrators a
clear understanding of Sahara's usage and needs. This should also note the
impact of Keystone configurations which do not provide default SQL identity
backends.

References
==========

Original bug report
https://bugs.launchpad.net/sahara/+bug/1321906


Keystone trust API reference
https://github.com/openstack/identity-api/blob/master/v3/src/markdown/identity-api-v3-os-trust-ext.md

