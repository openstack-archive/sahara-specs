..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

============================
Deprecation of Direct Engine
============================

https://blueprints.launchpad.net/sahara/+spec/deprecate-direct-engine

Currently Sahara has two types of infrastructure engines. First is Direct
Infrastructure Engine and second is Heat Infrastructure Engine. This spec
proposes deprecating the Direct Engine.

Problem description
===================

Each time when Sahara start support new feature, Sahara should support it in
both engines. So, it became much harder to support both versions of engines,
because in case of Direct Engine it would need to have duplication of work,
which is already done in Heat.

Proposed change
===============

It's proposed to deprecate Direct Engine in Liberty release, but it will be
available to use. After merging this spec Direct Engine should be ``freezed``
for new feautures which will be added in Liberty. It will be opened for
fixes of ``High`` and ``Critical`` bugs. We should make Heat Engine used by
default in Sahara.

This change will allow to switch most testing jobs in Sahara CI
to use Heat Engine instead of Direct Engine.

In M release we should remove all operations from direct engine.
After that the only operation which can be done with direct-engine-created
cluster is the cluster deletion. We should rewrite cluster deletion behavior
to support deletion direct-engine-created cluster via Heat Engine. Now heat
engine removes cluster from database but doesn't remove all cluster elements
(for example, instances).

Alternatives
------------

Sahara can continue support of both engines.

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

Deployers should switch to use Heat Engine instead of Direct Engine.

Developer impact
----------------

New features, which impacts infrastructure part of Sahara, should be supported
only in Heat Engine.

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

Other contributors:
  None

Work Items
----------

This change will require following changes:

* Add deprecation warnings at sahara startup.
* Mark Heat Engine as default in Sahara.
* Document deprecation of Direct Engine.

Dependencies
============

None

Testing
=======

This change require manual testing of deletion direct-engine-created cluster
after switch to heat engine.

Documentation Impact
====================

Need to document that Direct Engine became deprecated.

References
==========

None
