..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

================================================
Remove support of Hadoop 2.3.0 in Vanilla plugin
================================================


https://blueprints.launchpad.net/sahara/+spec/drop-hadoop-2-3-support


At the ATL Design summit it was decided to have 1 OpenStack release cycle
management for support of previous Hadoop versions in Vanilla plugin.
So it's time to remove 2.3.0 plugin.


Problem description
===================

Current Sahara code contains Vanilla Hadoop 2.3.0 in sources.

Proposed change
===============

The proposed change is to remove all Hadoop 2.3 and any related mentions
from Sahara code and subprojects.

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

Users will not be able to deploy Hadoop 2.3

Deployer impact
---------------

None

Developer impact
----------------

None

Sahara-image-elements impact
----------------------------

Hadoop 2.3 related elements should be removed

Sahara-dashboard / Horizon impact
---------------------------------

None

Implementation
==============

Assignee(s)
-----------

Primary Assignee:
Sergey Reshetnyak (sreshetniak)

Other Assignees:
Sergey Lukjanov (slukjanov) - dib cleanup

Work Items
----------

* Remove Vanilla plugin 2.3.0 from Sahara sources
* Remove DIB stuff related to 2.3.0
* Clear unit/integration tests
* Replace EDP examples with newest version of hadoop-examples
* Documentation changes

Dependencies
============

None

Testing
=======

None

Documentation Impact
====================

* Documentation must to be updated accordingly

References
==========

None
