..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===================================
Decompose cluster template for Heat
===================================

https://blueprints.launchpad.net/sahara/+spec/heat-template-decomposition

Currently Sahara creates one large template with a lot of copy-paste
resources. Heat features like `template composition
<http://docs.openstack.org/developer/heat/template_guide/composition.html>`_
could help to move all composition work from Sahara to Heat. This will also
allow sahara to have individual cluster parts as separate templates and insert
them as resources (in comparison to current text manipulations).

Problem description
===================

Currently Sahara serializes cluster resources as text to heat template. There
are several issues with this approach:

1. Code duplication. If a node group contains 10 instances the template will
   contain all instance-dependent resources 10 times.
2. No code validation. There is no guarantee that resulting template will be
   syntactically correct (Sahara treats it as text). Missing comma in one
   resource could influence the other resource.
3. Not Sahara's work. Sahara micro-manages the process of infrastructure
   creation. This is Heat's job.

Proposed change
===============

Use `OS::Heat::ResourceGroup <http://docs.openstack.org/hot-reference/content/OS__Heat__ResourceGroup.html>`_
for resources inside node group. Each node group will contain only one
resource group and specify number of instances needed. Each individual
instance of resource group will contain all resources needed for a
corresponding sahara instance (nova server, security group, volume,
floating ip, etc.).

This change will also prepare ground for node group auto-scaling feature.

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

Resulting Sahara stack in Heat will contain nested stacks.

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

* Add ability to generate separate ResourceGroup for a single instance of node
  group
* Switch template for node groups to ResourceGroup with specified count of
  instances
* Update unit tests
* Make sure that Sahara with heat engine still works for all supported
  configurations

Dependencies
============

None.

Testing
=======

Manually. CI will also cover changes in heat.

Documentation Impact
====================

None.

References
==========

None.