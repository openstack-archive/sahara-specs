..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===============================
add cdh 5.5 support into sahara
===============================

https://blueprints.launchpad.net/sahara/+spec/cdh-5-5-support

This specification proposes to add CDH 5.5 plugin with Cloudera Distribution
of Hadoop and Cloudera Manager in Sahara.

Problem description
===================

Now we have already supported plugins for CDH 5.3.0 and 5.4.0 versions
in liberty. With the release of the CDH 5.5.0 by Cloudera, we can add the new
version support into Sahara.

Proposed change
===============

Since we already support 5.4.0, we can follow the current implemention to
avoid too many changes. We must guarantee all the services supported in
previous version work well in 5.5.0. Supporting for new services in CDH 5.5.0
will be discussed later.

Cloudera starts to support ubuntu 14.04 in CDH 5.5.0. So we decide to provide
Ubuntu 14.04 as other plugins do. And the building for Ubuntu 14.04 image with
CDH 5.5 should aslo be supported in sahara-image-elements project. CentOS 6.5
will still be supported.

Due to the refactoring for previous CDH plugin versions, we should not merge
patches related with 5.5.0 until all the refactoring patches are merged.

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

None

Developer impact
----------------

None

Sahara-image-elements impact
----------------------------

Sahara-image-elements support for CDH 5.5.0 need to be done.

Sahara-dashboard / Horizon impact
---------------------------------

None

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  jxwang92 (Jaxon Wang)

Other contributors:
  None

Work Items
----------

The work items will be:

* Add python codes in sahara/sahara/plugins/cdh/v5_5_0.
* Add service resource files in sahara/sahara/plugins/cdh/v5_5_0/resources.
* Add test cases including unit and scenario.
* Test and evaluate the change.

Dependencies
============

None

Testing
=======

Follow what exsiting test cases of previous version do.

Documentation Impact
====================

Cloudera Plugin doc needs some little changes.
http://docs.openstack.org/developer/sahara/userdoc/cdh_plugin.html

References
==========

None
