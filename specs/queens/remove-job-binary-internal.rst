==========================================
Remove Job Binary Internal
==========================================

https://blueprints.launchpad.net/sahara/+spec/remove-job-binary-internal

Job binary internal is not needed now, since swift and manila
are available (and possibly other storage options in the future)
and these are more suitable options for storage.

Problem description
===================

Job binary internal is a job binary storage option that is kept in
Sahara's internal database, this option should not be available
anymore since there are better storage options (swift, manila, ...).

Besides that, Sahara's internal database should be used only to
Sahara's data storage. Allowing job binaries to be stored in
Sahara's database isn't only unneeded, but also a possible source
of problems, since it increases the size of the database. It also opens
a loophole for free storage. It's definitely the wrong tool for the job.

Also, it's important to notice that this change is related to
APIv2 and should not break APIv1.

Proposed change
===============

This change proposes to remove job binary internal in
favour of other storage options. Planning to deprecate
it when APIv2 is stable, in tandem with a deprecation of APIv1. Both APIv1
and JBI will be fully removed together after APIv2 has been stable for long
enough.

This change can be divided in the patches below.

* remove job binary internal from APIv2
* remove internal code that deals with job binary internal
  (possibly a few patches)
* remove job binary internal from database
* remove job binary internal from saharaclient
* remove job binary internal option from Horizon
  (sahara-dashboard)
* update documentation involving job binary internal

Alternatives
------------

Another option for this change would be not remove job
binary internal, which could bring future problems to Sahara.

Data model impact
-----------------

Job binary internal will be removed from the data model.
This will require a database migration.

REST API impact
---------------

Job binary internal related requests must be removed
only in APIv2.

Other end user impact
---------------------

Job binary internal option should not be available
through Horizon.

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

Job binary internal option should not be available
through Horizon.

Implementation
==============

Assignee(s)
-----------

Primary assignee: mariannelinharesm

Work Items
----------

* remove job binary internal from APIv2
* remove internal code that deals with job binary internal
  (possibly a few patches)
* remove job binary internal from database
* remove job binary internal from saharaclient
* remove job binary internal option from Horizon
  (sahara-dashboard)
* update documentation involving job binary internal
* (all but first and last steps are done in tandem with APIv1 removal)

Dependencies
============

None

Testing
=======

None

Documentation Impact
====================

Update documentation to related to job binary internal:

* add a warning about job binary internal's deprecation
  whenever APIv2 becomes stable and default.

References
==========

[0] http://eavesdrop.openstack.org/meetings/sahara/2017/sahara.2017-04-06-14.00.log.txt
[1] http://eavesdrop.openstack.org/meetings/sahara/2017/sahara.2017-03-30-18.00.log.txt

