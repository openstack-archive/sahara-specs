..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=====================================
[EDP] Add Spark Shell Action job type
=====================================

https://blueprints.launchpad.net/sahara/+spec/edp-add-spark-shell-action

The EDP Shell action job type allows users to run arbitrary
shell scripts on their cluster, providing a great deal of flexibility
to extend EDP functionality without engine changes or direct cluster
interface. This specification proposes the addition of this job type
for the Spark engine.

A fuller explication of the benefits of this feature can be found in
the edp-add-oozie-shell-action_ specification, which need not be
repeated here.

.. _edp-add-oozie-shell-action: http://specs.openstack.org/openstack/sahara-specs/specs/kilo/edp-add-oozie-shell-action.html


Problem description
===================

While the Oozie engine now supports Shell actions, Spark users do not
presently have access to this job type. Its addition would allow the
creation of cluster maintenance tools, pre- or post-processing jobs
which might be cumbersome to implement in Spark itself, the retrieval
of data from filesystems not supported as Sahara data sources, or any
other use case possible from a shell command.


Proposed change
===============

From an interface standpoint, the Spark Shell action implementation will
follow the Oozie Shell action implementation almost precisely:

* Shell jobs will require a single script binary in ``mains``, which will be
  pushed to the cluster's master node and executed.
* Shell jobs will optionally permit any number of file binaries to be
  passed as ``libs``, which will be placed in the script's working directory
  and may be used by the script as it executes.
* ``configs`` will be permitted to allow Sahara EDP-internal features
  (``substitute_data_source_for_uuid`` and ``subsitute_data_source_for_name``
  will be implemented for this job type, as they are for Oozie Shell actions.)
* ``params`` key-value pairs will be passed to the script as environment
  variables (whether these are passed into a remote ssh client or injected
  into the script itself is left to the discretion of the implementer.)
* ``args`` will be passed to the script as positional command-line arguments.
* The Shell engine for Spark will store files as the main Spark engine does,
  creating a directory under /tmp/spark-edp/``job_name``/``job_execution_id``,
  which will contain all required files and the output of the execution.
* The Spark Shell engine will reuse the ``launch_command.py`` script (as used
  by the main Spark engine at this time,) which will record childpid, stdout,
  and stderr from the subprocess for record-keeping purposes.

Spark Shell actions will differ in implementation from Oozie Shell actions
in the following ways:

* As Spark jobs and Shell actions which happen to be running on a Spark
  cluster differ quite entirely, the Spark plugin will be modified to contain
  two separate engines (provided via an extensible strategy pattern based on
  job type.) Sensible abstraction of these engines is left to the discretion
  of the implementer.
* ``configs`` values which are not EDP-internal will not be passed to the
  script by any means (as there is no intermediary engine to act on them.)
* Spark Shell actions will be run as the image's registered user, as Spark
  jobs are themselves. As cluster and VM maintenance tasks are part of the
  intended use case of this feature, allowing sudo access to the VMs is
  desirable.

Alternatives
------------

Do nothing.

Data model impact
-----------------

None.

REST API impact
---------------

No additional changes after merge of the Oozie Shell action job type
implementation.

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

No additional changes after merge of the Shell action job type UI
implementation.


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  sgotliv

Other contributors:
  egafford

Work Items
----------

* Add a Shell engine to the Spark plugin, and refactor this plugin to provide
  an appropriate engine branching on job type.
* Add an integration test for Spark Shell jobs (as per previous plugin-
  specific Shell job tests).
* Update the EDP documentation to specify that the Spark plugin supports the
  Shell job type.
* Verify that the UI changes made for Oozie Shell jobs are sufficient to
  support the Shell job type in the Spark case (as is anticipated).


Dependencies
============

This change builds on the change `[EDP] Add Oozie Shell Job Type`_.

.. _[EDP] Add Oozie Shell Job Type: https://review.openstack.org/#/c/159920/


Testing
=======

* Unit tests to cover the Spark Shell engine and appropriate engine selection
  within the plugin.
* One integration test to cover running of a simple shell job through the
  Spark plugin.


Documentation Impact
====================

The EDP sections of the documentation need to be updated.


References
==========

None.