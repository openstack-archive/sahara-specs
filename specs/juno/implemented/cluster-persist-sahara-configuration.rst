..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

================================================
Store Sahara configuration in cluster properties
================================================

https://blueprints.launchpad.net/sahara/+spec/cluster-persist-sahara-configuration


It is important to know Sahara version and configuration for proper cluster
migration and preventing dangerous operations.

Problem description
===================

Now Sahara has no way to know conditions when cluster was created. Cluster
operations after Openstack upgrade or switching infrastructure engine could be
dangerous and cause data loss.

Proposed change
===============

Store main Sahara properties (version, type and version of infrastructure
engine, etc) to cluster DB object. This will allow to prevent dangerous
operations and notify user gracefully.

Alternatives
------------

Don't store any information about Sahara settings. Always assume that settings
didn't change and we have current or previous release of OpenStack.

Data model impact
-----------------

New field in cluster object. Probably this should be dictionary with defined
keys.

REST API impact
---------------

We can expose information about Sahara settings to user. Or not.

Other end user impact
---------------------

Some error messages could become more descriptive.

Deployer impact
---------------

None

Developer impact
----------------

More options to handle errors.

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
  alazarev

Work Items
----------

Implement change

Dependencies
============

None

Testing
=======

Manual

Documentation Impact
====================

None

References
==========

None