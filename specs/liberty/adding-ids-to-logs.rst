..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=========================================================
Adding cluster/instance/job_execution ids to log messages
=========================================================

https://blueprints.launchpad.net/sahara/+spec/logs-improvement

This specification proposes to add more information to Sahara logs.

Problem description
===================

Looking at some Sahara logs it is difficult to determine what
cluster/instance/job_execution to which they refer.

Extra information should be added:

* logs associated with cluster creation/scaling/deletion should contain
  cluster id;
* logs associated with job execution/canceling/deletion should contain
  job execution id;
* logs associated with operations executed on specific instance should
  contain id of this instance.

Proposed change
===============

Information can be added to context resource_uuid field and then can be
used by ContextAdapter in openstack.common.log for a group of logs.

This change requires additional saving of context in
openstack.common.local.store to access context from openstack.common.log

We need to set cluster id and job execution id only once. So, it could be done
with 2 methods that will be added to sahara.context:
set_current_cluster_id(cluster_id) and set_current_job_execution_id(je_id)

Additionally, instances and their ids are changing during the thread. So,
instance id should be set only when operation executes on this instance.
It will be provided by class SetCurrentInstanceId and will be used with
wrapping function set_current_instance_id(instance_id) this way:

.. sourcecode:: python

    with set_current_instance_id(instance_id):

..

Code inside "with" statement will be executed with new context (which
includes instance id in resource_uuid field) but outside of it context will
stay the same.

If instance and cluster specified, log message will looks like:

.. sourcecode:: console

    2014-12-22 13:54:19.574 23128 ERROR sahara.service.volumes [-] [instance:
    3bd63e83-ed73-4c7f-a72f-ce52f823b080, cluster: 546c15a4-ab12-4b22-9987-4e
    38dc1724bd] message

..

If only cluster specified:

.. sourcecode:: console

    2014-12-22 13:54:19.574 23128 ERROR sahara.service.volumes [-] [instance:
    none, cluster: 546c15a4-ab12-4b22-9987-4e38dc1724bd] message

..

If job execution specified:

.. sourcecode:: console

    2014-12-22 13:54:19.574 23128 ERROR sahara.service.edp.api [-] [instance:
    none, job_execution: 9de0de12-ec56-46f9-80ed-96356567a196] message

..

Field "instance:" is presented in every message (even if it's not necessary)
because of default value of instance_format='[instance: %(uuid)s]  '
that cannot be fixed without config changing.

After implementation of this changes, Sahara log messages should be checed and
fixed to avoid information duplication.

Alternatives
------------

Information can be added manually to every log message.

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
  apavlov-n

Work Items
----------

* Adding ability to access context from openstack.common.log;
* Adding information about cluster/instance/job execution ids to context;
* Fixing log messages to avoid information duplication.

Dependencies
============

None

Testing
=======

None

Documentation Impact
====================

None

References
==========

None
