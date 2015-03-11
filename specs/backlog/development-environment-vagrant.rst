..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===============================
Development Environment Vagrant
===============================


https://blueprints.launchpad.net/sahara/+spec/development-environment-vagrant

The purpose of this blueprint is to ease the deployment of a
development environment for Sahara. It could be used by two
types of users:

* Developers requiring a development environment very fast
* End-users that want to try latest version of Sahara
  without any configuration constraints.


Problem description
===================

The setup of a development environment may be seen as a constraint for
new comers.

Proposed change
===============

The proposed solution is to use ``Vagrant`` in order to spawn a VM which
installs ``devstack`` with Sahara services.
Vagrant will isolate dependencies and the configuration in a single
disposable consistent environment. Hence you won't break anything on
your own host by trying to install Sahara.
You will also be able to use the Vagrant openstack provider which
will be useful to spawn your development environment in your
OpenStack Cloud.
A Vagrantfile will be added in sahara repository so users will just
have to clone the repository and execute the command ``vagrant up``.


Alternatives
------------

Let the user create a VM on their own, do the install and configuration
of their devstack without any tool or resources.

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

Primary Assignee:
Pierre Padrixe (stannie)

Work Items
----------

Add Vagrantfile in sahara repository


Dependencies
============

None

Testing
=======

None

Documentation Impact
====================

The usage of Vagrant with Sahara will be documented


References
==========

None
