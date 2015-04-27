..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=========
Storm EDP
=========

https://blueprints.launchpad.net/sahara/+spec/storm-scaling

This blueprint aims to implement Scaling for Storm.

Problem description
===================

Storm plugin in sahara doesn't have the scaling option implemented yet. This
feature is one of the major attractions to building cluster using sahara.

Proposed change
===============

The implementation of the scaling feature following the implementation from
Spark plugin.

The implementation will allow users to:

* Scale up a cluster
* Scale down a cluster

Storm is a fairly easy tool to scale. Since it uses Zookeeper as a
configuration manager and central point of communication, a new node just
needs to configure itself to communicate with the Zookeeper machine and the
master node will find the new node. One important point that needs to be taken
in consideration is the Storm rebalance action. Once a new node is added, a
running topology will not be rescheduled to use the new instance. We are going
to call this rebalance action automatically so the user won't have to worry
about this call.

Alternatives
------------

None.

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

None.

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
  tellesmvn

Other contributors:

Work Items
----------

* Implement Storm Scaling feature
* Implement topology rebalance

Dependencies
============

None.


Testing
=======

Follow examples on scaling tests from other plugins to implement unit tests
for Storm scaling.

Documentation Impact
====================

None.

References
==========

None.
