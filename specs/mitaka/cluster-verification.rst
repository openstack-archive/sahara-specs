
..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

============================================
Implement Sahara cluster verification checks
============================================

https://blueprints.launchpad.net/sahara/+spec/cluster-verification

Now we don't have any check to take a look at cluster processes status through
Sahara interface. Our plans is to implement cluster verifications and ability
to re-trigger these verifications for the particular cluster.

Problem description
===================

Sahara doesn't have any check for cluster processes health monitoring.
Cluster can be broken or unavailable but Sahara will still think that
in ACTIVE status. This may result in end-users losses and so on.

Proposed change
===============

First of all let's make several important definitions in here.
A cluster health check is a perform a limited functionality check
of a cluster. For example, instances accessibility,
writing some data to ``HDFS`` and so on.
Each health check will be implemented as a class object,
implementing several important methods from an abstract base class(abc):

.. sourcecode:: python

    class ExampleHealthCheck(object):
        def __init__(self, cluster, *args, **kwargs):
            self.cluster = cluster
            # other stuff

        @abc.abstractmethod
        def available(self):
            # method will verify

        @abc.abstractmethod
        def check(self):
            # actual health will in here

        def execute(self):
            # based in availability of the check and results
            # of check will write correct data into database

..

The expected behaviour of ``check`` method
of a health check is return some important data in case when
everything is ok and in case of errors to raise an exception
with detailed info with failures reasons. Let's describe
important statuses of the health checks.

 * ``GREEN`` status: cluster in healthy and health check
   passed correctly. In description in such case
   we can have the following info: ``HDFS available for writing
   data`` or ``All datanodes are active and available``.
 * ``YELLOW`` status: ``YellowHealthError`` will be raised
   as result of the operation, it means that something is
   probably wrong with cluster. By the way cluster is still
   operable and can be used for running jobs. For example,
   in exception message we will see the following information:
   ``2 out of 10 datanodes are not working``.
 * ``RED`` status: ``RedHealthError`` will be raised in such
   case, it means that something is definitely wrong we the cluster
   we can't guarantee that cluster is still able to perform perations
   on it. For example, ``Amount of active datanodes is less then
   replication`` is possible messange in such case.
 * ``CHECKING`` state, health check is still running or
   was just created.

A cluster verification is a combination of the several cluster
health checks. Cluster verification will indicate the current status
for the cluster: ``GREEN``, ``YELLOW`` or ``RED``. We will
store the latest verification for the cluster in database. Also
we will send results of verifications to Ceilometer, to
view progress of cluster health.

Also there is an idea of running several jobs as part
of several health checks, but it will be too harmful for the
cluster health and probably will be done later.

Also we can introduce periodic task for refreshing health status
of the cluster.

So, several additional options in ``sahara.conf``
should be added in new section ``cluster_health_verification``:

 * ``enable_health_verification`` by default will
   be True, will allow disabling periodic cluster verifications
 * ``verification_period`` variable to define period
   between two consecutive health verifications in periodic tasks.
   By default I would suggest to run verification once in 10 minutes.

**Proposed checks**

This section is going to describe basic checks of functionalities
of clusters. Several checks will affect almost all plugins, few
checks will be specific only for the one plugin.

**Basic checks**:

There are several basic checks for all possible clusters
in here.

 * Check to verify all instances access. If some instances
   are not accessible we have ``RED`` state.
 * Check that all volumes mounted. If some volume is not
   mounted, we will have ``YELLOW`` state.

**HDFS checks**:

 * Check that will verify that namenode is working. Of course,
   ``RED`` state in bad case. Actually this will affects
   only vanilla plugin clusters and clusters deployed without
   HA mode in case of CDH and Ambari plugins.
 * Check that will verify amount of living datanodes. We will
   have ``GREEN`` status only when all datanodes are active,
   ``RED`` in case when amount of living datanodes are less
   then ``dfs.replicaton``, and ``YELLOW`` in all other cases.
 * Check to verify ability of cluster to write some data to
   ``HDFS``. We will have ``RED`` status if something is failed.
 * Amount of free space in HDFS. We will compare this value with
   reserved memory in HDFS,  and if amount of free space is less
   then provided value, check will be supposed to be failed.
   If check is not passed, we will have ``YELLOW`` state
   we will advice to scale cluster up with some extra datanodes (or
   clean some data). I think, we should not have any additional
   configuration options in here, just because this check
   will never report ``RED`` state.

**HA checks**:

 * ``YELLOW`` state in case when at least one stand-by service
   is working; and ``RED`` otherwise. Affects YARN and HDFS both.

**YARN checks**:

 * Resourcemanger is active. Obviously, ``RED`` state if something
   is wrong.
 * Amount of active nodemanagers. ``YELLOW`` state if something
   is not available, and ``RED`` if amout of live nodemanagers
   are less then ``50%``.

**Kafka check**:

 * Check that kafka is operable: create example topic, put
   several messages in topic, consuming messages. ``RED``
   state in case of something is wrong.

**CDH plugin check**:

This section is going to describe specific checks for CDH plugin.
For this checks we will need to extend current sahara's implementation of
``cm_api`` tool. There is an API methods to get current health
of the cluster. There are few examples of responses for yarn service.

There is the bad case example:

.. sourcecode:: console

  "yarn01": {
    "checks": [
      {
        "name": "YARN_JOBHISTORY_HEALTH",
        "summary": "GOOD"
      },
      {
        "name": "YARN_NODE_MANAGERS_HEALTHY",
        "summary": "CONCERNING"
      },
      {
        "name": "YARN_RESOURCEMANAGERS_HEALTH",
        "summary": "BAD"
      }
    ],
    "summary": "BAD"
  }

..

and good case example:

.. sourcecode:: console

  "yarn01": {
    "checks": [
      {
        "name": "YARN_JOBHISTORY_HEALTH",
        "summary": "GOOD"
      },
      {
        "name": "YARN_NODE_MANAGERS_HEALTHY",
        "summary": "GOOD"
      },
      {
        "name": "YARN_RESOURCEMANAGERS_HEALTH",
        "summary": "GOOD"
      }
    ],
    "summary": "GOOD"
  }

..


Based on responses above we will calculate health of
the cluster. Also possible states which Cloudera can return through API
are ``DISABLED`` when service was stopped and ``CONCERNING`` if something
is going to be bad soon. In this health check sahara's statuses
will be calculated based on the following table:

.. sourcecode:: console

 +--------------+--------------------------------+
 | Sahara state |   Cloudera state               |
 +--------------+--------------------------------+
 | GREEN        | All services GOOD              |
 +--------------+--------------------------------+
 | YELLOW       | At least 1 service CONCERNING  |
 +--------------+--------------------------------+
 | RED          | At least 1 service BAD/DISABLED|
 +--------------+--------------------------------+

..

Some additional information about Cloudera health checks are
in here: [0]

**Ambari plugin**:

Current ``HDP 2.0.6`` will support only basic verifications. The main
focus in here is to implement additional checks for the Ambari plugin.
There are several ideas of checks in Ambari plugin:

 * Ambari alerts verification. Ambari plugin have several alerts if something
   is wrong with current state of the cluster. We can get alerts through Ambari
   API. If we have at least one alert in here it's proposed to use ``YELLOW``
   status for the verification, and otherwise we will use ``GREEN`` status for
   that.
 * Ambari service checks verification. Ambari plugin have a bunch of services
   checks in here, which can be re-triggered by user through the Ambari API.
   These checks are well described in [1]. If at least one
   check failed, we will use ``RED`` status for that sutiation, otherwise
   it's nice to use ``GREEN``.

Alternatives
------------

All health checks can be disabled by the option.

Data model impact
-----------------

Graphical description of data model impact:

.. sourcecode:: console

 +----------------------------+    +-------------------------------+
 |       verifications        |    |           health_checks       |
 +----------------------------+    +-----------------+-------------+
 |    id      | Primary Key   |    | id              | Primary Key |
 +------------+---------------+    +-----------------+-------------+
 | cluster_id | Foreign Key   |  +-| verification_id | Foreign Key |
 +----------------------------+  | +-----------------+-------------+
 | created_at |               |  | | created_at      |             |
 +------------+---------------+  | +-----------------+-------------+
 | updated_at |               |  | | updated_at      |             |
 +------------+---------------+  | +-----------------+-------------+
 | checks     |               | <+ | status          |             |
 +------------+---------------+    +-----------------+-------------+
 | status     |               |    | description     |             |
 +------------+---------------+    +-----------------+-------------+
                                   | name            |             |
                                   +-----------------+-------------+
..

We will have two additional tables where we will store verifications
and health checks.

First table with of verifications will have following columns id,
cluster_id (foreign key), created_at, updated_at.

Also will be added new table to store health check results. This table
will have the following columns: id, verification_id,
description, status, created_at and updated_at.

We will have cascade relationship (checks) between cluster verifications and
cluster health checks to get correct access from health check to
cluster verification and vice versa. Also same relationship will be
between cluster and verification for same purpose.

Also to aggregation results of latest verification and disabling/enabling
verifications for particular cluster will be added the new column to
cluster model: ``verifications_status``. We will not use ``status``
for that purpose just to keep these two variables separately (we
already using status in many places in sahara).

For example of verifications:

1. One health check is still running:

.. sourcecode:: console

   "cluster_verification": {
     "id": "1",
     "cluster_id": "1111",
     "created_at": "2013-10-09 12:37:19.295701",
     "updated_at": "2013-10-09 12:37:19.295701",
     "status": "CHECKING",
     "checks": [
       {
         "id": "123",
         "created_at": "2013-10-09 12:37:19.295701",
         "updated_at": "2013-10-09 12:37:19.295701",
         "status": "GREEN",
         "description": "some description",
         "name": "same_name"
       },
       {
         "id": "221",
         "created_at": "2013-10-09 12:37:19.295701",
         "updated_at": "2013-10-09 12:37:19.295701",
         "status": "CHECKING",
         "description": "some description",
         "name": "same_name"
       },
     ]
   }
..

2. All health checks are completed but one was failed:

.. sourcecode:: console

  "cluster_verification": {
    "id": "2",
    "cluster_id": "1112",
    "created_at": "2013-10-09 12:37:19.295701",
    "updated_at": "2013-10-09 12:37:30.295701",
    "STATUS": "RED",
    "checks": [
      {
        ..
        "status": "RED",
        "description": "Resourcemanager is down",
        ..
      },
      {
        ..
        "status": "GREEN",
        "description": "HDFS is healthy",
      }
    ]
  }

..


REST API impact
---------------

Mechanism of receiving results of cluster verifications will be quite
simple. We will just use usual ``GET`` method for clusters.

So, the main API method will be the following:
``GET <tenant_id>/clusters/<cluster_id>``.
In such case, we will return detailed info of the cluster with verifications.

Example of response:

.. sourcecode:: console

 {
   "status": "Active",
   "id": "1111",
   "cluster_template_id": "5a9a09a3-9349-43bd-9058-16c401fad2d5",
   "name": "sample",
   "verifications_status": "RUNNING",
   ..
   "verification": {
     "id": "1",
     "cluster_id": "1111",
     "created_at": "2013-10-09 12:37:19.295701",
     "updated_at": "2013-10-09 12:37:19.295701",
     "checks": [
       {
         "id": "123",
         "created_at": "2013-10-09 12:37:19.295701",
         "updated_at": "2013-10-09 12:37:19.295701",
         "status": "GREEN",
         "description": "some description",
         "name": "same_name"
       },
       {
         "id": "221",
         "created_at": "2013-10-09 12:37:19.295701",
         "updated_at": "2013-10-09 12:37:19.295701",
         "status": "CHECKING",
         "description": "some description",
         "name": "same_name"
       },
     ]
   }
 }

..

For re-triggering to cluster verification, some additional
behaviour should be added to the following API method:

``PATCH <tenant_id>/clusters/<cluster_id>``

If the following data will be provided to this API method
we will re-trigger verification:

.. sourcecode:: console

 {
   'verification': {
     'status': 'START'
   }
 }

..

Start will be reject when verifications disabled for the
cluster or when verification is running on the cluster.

Also we can disable verification for particular cluster
to avoid unneeded noisy verifications until health issues are
fixed by the following request data:

.. sourcecode:: console

 {
   'verification': {
     'status': 'DISABLE'
   }
 }

..

And enable in case we need to enable health checks again. If
user is trying to disable verification only future verifications
will be disabled, so health checks still will be running.

If something additional will be added to this data we will mark
request as invalid. Also we will implement new validation methods
to deny verifications on cluster which already have one verification running.

Other end user impact
---------------------

Need to implement requests for run checks via python-saharaclient
and get their results.

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

Dashboard impact is need to add new tab in cluster details with results of
verifications.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  vgridnev

Other contributors:
  apavlov-n, esikachev

Work Items
----------

1. Implement basic skeleton for verifications (with base checks)
2. Python-saharaclient support addition
3. CLI support should be implemented
4. Implement tab with verification results to Horizon
5. Need to add new WADL docs with new api-method
6. All others checks should be implemented
7. Should be added support to scenario framework to allow
   re-triggering.
8. Implement sending history to Ceilometer.

Dependencies
============

None

Testing
=======

Feature will be covered by the unit tests, and manually.
New test commit (not for merging) will be added to show
that all verifications are passed (since we are at the middle
of moving scenario framework).

Documentation Impact
====================

Documentaion should updated with additional information
of ``How to`` repair issues described in the health check results.

References
==========

[0] http://www.cloudera.com/content/www/en-us/documentation/enterprise/latest/topics/cm_ht.html
[1] https://cwiki.apache.org/confluence/display/AMBARI/Running+Service+Checks
