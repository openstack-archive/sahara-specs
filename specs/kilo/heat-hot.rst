..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

============================
Migrate to HEAT HOT language
============================

https://blueprints.launchpad.net/sahara/+spec/heat-hot

This blueprint suggests to rewrite cluster template for Heat from JSON to HOT
language.

Problem description
===================

Heat supports two different template languages: YAML-based
`HOT <http://docs.openstack.org/developer/heat/template_guide/hot_guide.html>`_ templates
and JSON-based CFN templates (no documentation about it, only a number of
examples).

HOT is the de-facto main markup language for Heat. `Template Guide
<http://docs.openstack.org/developer/heat/template_guide/index.html>`_
recommends to use HOT and contains examples for HOT only.

CFN templates are supported mostly for compatibility with AWS CloudFormation.

Sahara historically uses CFN templates. Given that Sahara is an integrated
OpenStack project it would be nice to switch to HOT.

Proposed change
===============

There is no urgent need in switching to HOT. But it would be nice to be
compliant with current tendencies in community.

This spec suggests to use HOT template language for sahara heat templates.

This will require changes mostly in .heat resources. Code that generate
template parts on the fly should be changed too.

Having templates written on HOT will simplify implementation of new
heat-related features like `template decomposition
<https://blueprints.launchpad.net/sahara/+spec/heat-template-decomposition>`_.

Alternatives
------------

Do not change anything.

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
  alazarev (Andrew Lazarev)

Other contributors:
  None

Work Items
----------

* Change all .heat files used by Sahara
* Update code that generates parts of template
* Update unit tests
* Make sure that sahara with heat engine still works in all supported
  configurations

Dependencies
============

None

Testing
=======

Mostly manually. CI should also cover heat changes.

Documentation Impact
====================

None.

References
==========

* http://docs.openstack.org/developer/heat/template_guide/hot_guide.html