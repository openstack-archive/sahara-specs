..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=====================================
Support of distributed periodic tasks
=====================================

https://blueprints.launchpad.net/sahara/+spec/distributed-periodics

This specification proposes to add distributed periodic tasks.

Problem description
===================

Currently periodic tasks are executed on each engine simultaneously, so that
the amount of time between those tasks can be very small and load between
engines is not balanced. Distribution of periodic tasks between engines can
deal with both problems.

Also currently we have 2 periodic tasks that make clusters cleanup. Clusters
terminations in these tasks are performed in cycle with direct termination
calls (bypassing OPS). This approach leads to cessation of periodic tasks on
particular engine during cluster termination (it harms even more if some
problems during termination occurs).

Moreover distribution of periodic tasks is important for peridic health checks
to prevent extra load on engines.


Proposed change
===============

This change consists of two things:

* Will be added ability to terminate clusters in periodic tasks via OPS;
* Will be added support of distributed periodic tasks.

Distributed periodic tasks will be based on HashRing implementation and Tooz
library that provides group membership support for a set of backends [1].
Backend will be configured with ``periodic_coordination_backend`` option.

There will be only one group called ``sahara-periodic-tasks``. Once a group is
created, any coordinator can join the group and become a member of it.

As an engine joins the group and builds HashRing, hash of its ID is being
made, and that hash determines the data it's going to be responsible for.
Everything between this number and one that's next in the ring and that belongs
to a different engine, is now belong to this engine.

The only remaining problem is that some engines will have disproportionately
large space before them, and this will result in greater load on them. This can
be ameliorated by adding each server to the ring a number of times in different
places. This is achieved by having a replica count (will be 40 by default).

HashRing will be rebuilt before execution of each periodic task to reflect
actual state of coordination group.

``sahara.service.coordinator.py`` module and two classes will be added:

* ``Coordinator`` class will contain basic coordination and grouping methods:

  .. sourcecode:: python

    class Coordinator(object):
        def __init__(self, backend_url):
            ...

        def heartbeat(self):
            ...

        def join_group(self, group_id):
            ...

        def get_members(self, group_id):
            ...

* ``HashRing`` class will contain methods for ring building and subset
  extraction:

  .. sourcecode:: python

    class HashRing(Coordinator):
        def __init__(self, backend_url, group_id, replicas=100):
            ...

        def _build_ring(self):
            ...

        def get_subset(self, objects):
            ...

Now we have 4 periodic tasks. For each of them will be listed what exactly
will be distributed:

* ``update_job_statuses``
  Each engine will get a list of all job executions but will update statuses
  for only a part of them according to a hash values of their IDs.
* ``terminate_unneeded_transient_clusters``
  Each engine will get a list of all clusters, check part of them and request
  termination via OPS if needed.
* ``terminate_incomplete_clusters``
  Same as for ``terminate_unneeded_transient_clusters``.
* ``check_for_zombie_proxy_users``
  Each engine will get a list of users but will check if it is zombie or not
  only for part of them.

Also we will have a periodic task for health checks soon. Health check task
will be executed on clusters and this clusters will be split among engines in
the same way as job executions for ``update_job_statuses``.

If a coordination backend is not provided during configuration, periodic
tasks will be launched in an old-fashioned way and HashRing will not be built.

If a coordination backend is provided, but configured wrong or not accessible,
engine will not be started with corresponded error.

If a connection to the coordinator will be lost, periodic tasks will be
stopped. But once connection is established again, periodic tasks will be
executed in the distributed mode.

In order to keep the connection to the coordination server active,
``heartbeat`` method will be called regularly (every ``heartbeat_timeout``
seconds) in a separate thread.

Configurable number of threads (each thread will be a separate member of a
group) performing periodic tasks will be launched.


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

Coordination backend should be configured.

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
  apavlov-n

Work Items
----------

* Adding ability to terminate clusters in periodic tasks via OPS;
* Implementing HashRing for distribution of periodic tasks;
* Documenting changes in periodic tasks configuration.
* Adding support of distributed periodics to devstack with ZooKeeper as a
  backend

Dependencies
============

Tooz package [2]

Testing
=======

Unit tests, enabling distributed periodics in intergration tests with one of
the supported backends (for example, ZooKeeper) and manual testing for all
available backends [1] supported by Tooz library.

Documentation Impact
====================

Sahara REST API documentation in api-ref will be updated.

References
==========

[1]: http://docs.openstack.org/developer/tooz/compatibility.html#driver-support

[2]: https://pypi.python.org/pypi/tooz