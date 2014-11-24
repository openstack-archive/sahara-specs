..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==============================================
Storage of recently logged events for clusters
==============================================

https://blueprints.launchpad.net/sahara/+spec/event-log

This specification proposes to add event logs assigned to cluster.

Problem description
===================

It will be more user friendly have event log assigned to cluster.
In this case users will have the ability to see the steps performed to deploy
a cluster. If there is an issue with their cluster, users will be able to see
the reasons for the issues in the UI and won't be required to read the Sahara
logs.

Proposed change
===============

The new feature will provide the following ability:

* For each cluster there will be an event log assigned to it. The deployer
  will have the ability to see it in Horizon. In that case the user will have
  the ability to see all the steps for the cluster deploying.
* The deployer will have the ability to see the current progress of cluster
  provisioning.

Alternatives
------------

None

Data model impact
-----------------

This change will require to write event log messages to the database.
It's good idea to store events messages in database in similar
manner, in which we store cluster data, node groups data and so on.

Plugins should provide a list of provisioning steps and be able to report
status of the current step. All steps should be performed in linear series,
and we will store events only for the current step. All completed steps should
have the duration time stored in the database. There are no reasons to store
events for successfully completed steps, so they will be dropped periodically.

If an error occurs while provisioning cluster we will have error events saved
for the current step. Also we should store for each error event error_id,
which would help for admins to determine reasons of failures in sahara logs.

We should have new a database object called ClusterEvent, which will have the
following fields:

* node_group_id;
* instance_id;
* instance_name;
* event_info;
* successful;
* provision_step_id;
* id;
* created at;
* updated at;

Also we should have a new database object called ClusterProvisionStep,
which will have the following fields:

* id;
* cluster_id;
* step_name;
* step_type;
* completed;
* total;
* successful;
* started_at;
* completed_at;
* created_at;
* updated_at;

Fields ``step_name`` and ``step_type`` will contain detail info about step.
``step_name`` will contain description of the step, for example
``Waiting for ssh`` or ``Launching instances``. ``step_type`` will
contain info about related process of this step. For example, if we
creating new cluster this field will contain ``creating`` and for
scaling some cluster this field will contain ``scaling``.
So, possible values of this field will be ``creating``, ``scaling``,
``deleting``. Also we should add ability to get main provisioning steps from
Plugin SPI for each ``step_type`` as dictionary. For example, expected
return value:

.. sourcecode:: console

   {
      "creating": [
          "Launching instances",
          "Waiting for ssh",
          ....
      ]
      "scaling": [
          ....
      ]
      "deleting": [
          ....
      ]
   }

..

Cluster should have new field:
* provisioning_progress

This field will contain list with provisioning steps, which should provide
ability to get info about provisioning steps from cluster. We should update
this list with new steps every time we start new process with cluster
(creating/scaling/deleting). Provision steps should updated both from plugin
and infra, because some steps are same for all clusters.

REST API impact
---------------

Existing GET request for a cluster should be updated with completed steps
info, and short info for the current step. For example, we will have
following response: "Launching instances completed 1000 out of 1000 in 10
minutes", "Trying ssh completed: 59 out of 1000". Also response should be
sorted by increasing of value ``created_at``.

.. sourcecode:: console

   {
      "cluster": {
          "status": "Waiting",
          ....
          "provisioning_progress": [
             {
               "id": "1",
               "cluster_id": "1111",
               "step_name": "Launching instances",
               "step_type": "creating",
               "completed": 1000,
               "total": 1000,
               "successful": "True",
               "created_at": "2013-10-09 12:37:19.295701",
               "started_at": 36000000,
               "completed_at": 18000000,
               "updated_at": "2013-10-09 12:37:19.295701",
             },
             {
               "id": "2",
               "cluster_id": "1111",
               "step_name": "Waiting for ssh",
               "step_type": "creating",
               "completed": 59,
               "total": 1000,
               "successful": None,
               "created_at": "2013-10-09 12:37:19.295701",
               "started_at": 18000000,
               "completed_at": None,
               "updated_at": "2013-10-09 12:37:19.295701",
             }
          ]
          ....
      }
   }

..

In case of errors:

.. sourcecode:: console

   {
      "cluster": {
          "status": "Waiting",
          ....
          "provisioning_progress": [
             {
               "id": "1",
               "cluster_id": "1111",
               "step_name": "Launching instances",
               "step_type": "creating",
               "completed": 1000,
               "total": 1000,
               "successful": "True",
               "created_at": "2013-10-09 12:37:19.295701",
               "started_at": 36000000,
               "completed_at": 18000000,
               "updated_at": "2013-10-09 12:37:19.295701",
             },
             {
               "id": "2",
               "cluster_id": "1111",
               "step_name": "Waiting for ssh",
               "step_type": "creating",
               "completed": 59,
               "total": 1000,
               "successful": False,
               "created_at": "2013-10-09 12:37:19.295701",
               "started_at": 18000000,
               "completed_at": None,
               "updated_at": "2013-10-09 12:37:19.295701",
             }
          ]
          ....
      }
   }

..

Also in these cases we will have events stored in database from which we can
debug cluster problems.
Because first steps of cluster provision are same, then for these steps
infra should update ``provisioning_progress`` field. Also for all
plugin-related steps plugin should update ``provisioning_progress`` field.
So, new cluster field should be updated both from infra and plugin.

New endpoint should be added to get details of the current provisioning step:
GET /v1.1/<tenant_id>/clusters/<cluster_id>/progress

The expected response should looks like:

.. sourcecode:: console

   {
        "events": [
              {
                 'node_group_id': "ee258cbf-4589-484a-a814-81436c18beb3",
                 'instance_id': "ss678cbf-4589-484a-a814-81436c18beb3",
                 'instance_name': "cluster-namenode-001",
                 'provisioning_step_id': '1',
                 'event_info': None,
                 'successful': True,
                 'id': "ss678cbf-4589-484a-a814-81436c18eeee",
                 'created_at': "2014-10-29 12:36:59.329034",
              },
              {
                 'cluster_id': "d2498cbf-4589-484a-a814-81436c18beb3",
                 'node_group_id': "ee258www-4589-484a-a814-81436c18beb3",
                 'instance_id': "ss678www-4589-484a-a814-81436c18beb3",
                 'instance_name': "cluster-datanode-001",
                 'provisioning_step_id': '1',
                 'event_info': None,
                 'successful': True,
                 'id': "ss678cbf-4589-484a-a814-81436c18eeee",
                 'created_at': "2014-10-29 12:36:59.329034",
              },
              {
                 'cluster_id': "d2498cbf-4589-484a-a814-81436c18beb3",
                 'node_group_id': "ee258www-4589-484a-a814-81436c18beb3",
                 'instance_id': "ss678www-4589-484a-a814-81436c18beb3",
                 'instance_name': "cluster-datanode-001",
                 'provisioning_step_id': '2',
                 'event_info': "Trying to access failed: reason in sahara logs
                               by id ss678www-4589-484a-a814-81436c18beb3",
                 'successful': False,
                 'id': "ss678cbf-4589-484a-a814-81436c18eeee",
                 'created_at': "2014-10-29 12:36:59.329034",
              },
        ]
   }

..

Event info for the failed step will contain the traceback of an error.

Other end user impact
---------------------

None

Deployer impact
---------------

This change will takes immediate effect after it is merged.
Also it is a good idea to have ability to disable event log from
configuration.

Developer impact
----------------

None

Sahara-image-elements impact
----------------------------

None

Sahara-dashboard / Horizon impact
---------------------------------

This change will add section in Horizon at page with event logs
/data_processing/clusters/cluster_id.
At this page it will be possible to see main provisioning steps,
and current progress of all of it.
Also we would have an ability to see events of current provisioning
step. In case of errors we will be able to see all events of the current
step and main reasons of failures.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
 vgridnev

Other contributors:
 slukjanov, alazarev, nkonovalov

Work Items
----------

This feature require following modifications:
 * Add ability to get info about main steps of provisioning cluster from
   plugin;
 * Add ability to view progress of current provisioning step;
 * Add ability to specify events to current cluster and current step;
 * Add periodic task to erase redundant events from previous step;
 * Add ability to view events about current step of cluster provisioning;
 * Sahara docs should be updated with some use cases of this feature;
 * Saharaclient should be modified with new REST API feature;
 * New cluster tab with events in Horizon should be implemented;
 * Add unit test to test new features of events.

Dependencies
============

Depends on OpenStack requirements

Testing
=======

As written in Work Items section this feature will require unit tests

Documentation Impact
====================

As written in Work Items section this feature will require docs updating
with some use cases of feature

References
==========

