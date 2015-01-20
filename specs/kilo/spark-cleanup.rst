..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==============================================
Spark Temporary Job Data Retention and Cleanup
==============================================

https://blueprints.launchpad.net/sahara/+spec/spark-cleanup

Creates a configurable cron job at cluster configuration time to clean up data
from Spark jobs, in order to ease maintenance of long-lived clusters.

Problem description
===================

The current Spark plugin stores data from any job run in the /tmp directory,
without an expiration policy. While this is acceptable for short-lived
clusters, it increases maintenance on long-running clusters, which are likely
to run out of space in time. A mechanism to automatically clear space is
needed.

Proposed change
===============

On the creation of any new Spark cluster, a script (from
sahara.plugins.spark/resources) will be templated with the following
variables (which will be defined in Spark's config_helper module and thus
defined per cluster):

* Minimum Cleanup Seconds
* Maximum Cleanup Seconds
* Minimum Cleanup Megabytes

That script will then be pushed to /etc/hadoop/tmp_cleanup.sh. In the
following cases, no script will be pushed:

1) Maximum Cleanup Seconds is 0 (or less)
2) Minimum Cleanup Seconds and Minimum Cleanup Megabytes are both 0 (or less)

Also at cluster configuration time, a cron job will be created to run this
script once per hour.

This script will iterate over each extant job directory on the cluster; if it
finds one older than Maximum Cleanup Seconds, it will delete that directory.
It will then check the size of the set of remaining directories. If there is
more data than Minimum Cleanup Megabytes, then it will delete directories
older than Minimum Cleanup Seconds, starting with the oldest, until the
remaining data is smaller than Minimum Cleanup Megabytes or no sufficiently
aged directories remain.


Alternatives
------------

Any number of more complex schemes could be developed to address this problem,
including per-job retention information, data priority assignment (to
effectively create a priority queue for deletion,) and others. The above plan,
however, while it does allow for individual cluster types to have individual
retention policies, does not demand excessive maintenance or interface with
that policy after cluster creation, which will likely be appropriate for most
users. A complex retention and archival strategy exceeds the intended scope of
this convenience feature, and could easily become an entire project.

Data model impact
-----------------

None; all new data will be stored as cluster configuration.

REST API impact
---------------

None; operative Spark cluster template configuration parameters will be
documented the current interface allows this change.

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

None. (config_helper.py variables will be automatically represented in
Horizon.)

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  egafford

Work Items
----------

* Creation of periodic job
* Creation of deletion script
* Testing

Dependencies
============

None

Testing
=======

Because this feature is entirely Sahara-internal, and requires only a remote
shell connection to the Spark cluster (without which many, many other tests
would fail) I believe that Tempest tests of this feature are unnecessary. Unit
tests should be sufficient to cover this feature.

Documentation Impact
====================

The variables used to set retention policy will need to be documented.

References
==========

https://blueprints.launchpad.net/sahara/+spec/spark-cleanup
