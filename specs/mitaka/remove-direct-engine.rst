..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

============================
Deprecation of Direct Engine
============================

https://blueprints.launchpad.net/sahara/+spec/remove-direct-engine

Direct Infrastructure Engine was deprecated in Liberty and it's time to remove
that in Mitaka cycle.

Problem description
===================

We are ready to remove direct engine. Sahara have ability to delete clusters
created using direct engine, heat engine works nicely.

Proposed change
===============

First it's proposed to remove direct engine first. Then after the removal
we can migrate all direct-engine-jobs to heat engine. After that direct
engine will be completely removed.

Alternatives
------------

None

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

Deployers should switch to use Heat Engine instead of Direct Engine finally.

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
  vgridnev

Work Items
----------

This will require following changes:

* Remove direct engine from the codebase (with unit tests).
* Remove two gate jobs for direct engine.
* Document that Direct Engine was removed finally.

Dependencies
============

None

Testing
=======

None

Documentation Impact
====================

Need to document that Direct Engine was removed.

References
==========

None
