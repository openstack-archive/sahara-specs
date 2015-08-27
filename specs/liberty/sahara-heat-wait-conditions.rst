..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===========================
Heat WaitConditions support
===========================

https://blueprints.launchpad.net/sahara/+spec/sahara-heat-wait-conditions

Before Heat engine in Sahara Nova was continuously asked for fixed and
assigned floating IP and for active SSH connections to VMs. To get rid of
such polling mechanism suggested to use Heat WaitConditions feature.


Problem description
===================

Now Sahara checks instances availability via SSH. Wait Condition resource
supports reporting signals to Heat. We should report signal to Heat about
booting instance.


Proposed change
===============

Add WaitCondition resource to Sahara Heat template.

Alternatives
------------

Using SSH for polling instance accessible.

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

Primary assignee:
  sreshetniak

Work Items
----------

Add Wait Condition support to Sahara


Dependencies
============

WaitCondition requires pre-installed cloud-init.


Testing
=======

Need to add unit tests for this feature.
Integration tests will cover this feature.


Documentation Impact
====================

None


References
==========

http://docs.openstack.org/developer/heat/template_guide/openstack.html#OS::Heat::WaitCondition
