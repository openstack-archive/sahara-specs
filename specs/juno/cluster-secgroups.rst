..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==============================================================
Security groups management in Sahara
==============================================================


https://blueprints.launchpad.net/sahara/+spec/cluster-secgroups

It is not acceptable for production use to require default security group with
all ports open. Sahara need more flexible way to work with security groups.

Problem description
===================

Now Sahara doesn't manage security groups and use default security group for
instances provisioning.

Proposed change
===============

Solution will consist of several parts:

1) Allow user to specify list of security groups for each of node groups.
2) Add support of automatic security group creation. Sahara knows everything
   to create security group with required ports open. In the first iteration
   this will be security group with all exposed ports open for all networks.

Alternatives
------------

Creation of security groups by Sahara could be done in several ways. Ideally
Sahara should support separation between different networks and configuration
on what to allow and what is not.

Data model impact
-----------------

1) List of security groups need to be saved in each node group.
2) Flag indicating that one of security groups is created by Sahara
3) List of ports to be opened. It need to be stored somewhere to provide this
   information to provisioning engine.

REST API impact
---------------

Requests to create cluster, nodegroup, cluster template and nodegroup template
will be extended to receive security groups to use. Also option for
automatic security group creation will be added.


Other end user impact
---------------------

None

Deployer impact
---------------

In some cases there will be no need to configure default security group.

Developer impact
----------------

Plugin SPI will be extended with method to return required ports for node
group.

Sahara-image-elements impact
----------------------------

None

Sahara-dashboard / Horizon impact
---------------------------------

New field to select security group in all create screens.

Implementation
==============

Assignee(s)
-----------

Primary Assignee:
Andrew Lazarev (alazarev)

Work Items
----------

* Allow user to specify security groups for node group
* Implement ability of security group creation by Sahara

Both items require the following steps:

* Implement in both engines (heat and direct engine)
* Test for nova network and neutron
* Update documentation
* Update UI
* Create integration test

Dependencies
============

None

Testing
=======

Feature need to be covered by integration tests both for engine and UI.

Documentation Impact
====================

Feature need to be documented.


References
==========

None
