..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==================================================================
[EDP] Add options supporting DataSource identifiers in job_configs
==================================================================

https://blueprints.launchpad.net/sahara/+spec/edp-data-sources-in-job-configs

In some cases it would be convenient if users had a way to pass a DataSource
object as a job configuration value to take advantage of the data codified in
the object instead of copying and specifying that information manually.  This
specification describes options that allow users to reference DataSource
objects in configuration values, parameters, and arguments in a JobExecution
record.


Problem description
===================

With the exception of Java and Spark all job types in Sahara take a single
input DataSource object and a single output DataSource object. The DataSource
object ids are passed as part of a job execution request and Sahara configures
the job based on the information in the objects. Sahara assumes that jobs at
runtime will consume the path information using a fixed set of parameters and
configuration values. This works well in the general case for the constrained
job types supported by Oozie/hadoop (Hive, Pig, MapReduce) but there are cases
where DataSource objects currently may not be used:

* Java and Spark job types do not require DataSources since they have
  no fixed arg list. Currently input and output paths must be specified as URLs
  in the ``args`` list inside of ``job_configs`` and authentication configs
  must be manually specified.

* Hive, Pig, and MapReduce jobs which use multiple input or output paths or
  consume paths through custom parameters require manual configuration.
  Additional paths or special configuration parameters (ie anything outside
  of Sahara's assumptions) require manual specification in the ``args``,
  ``params``, or ``configs`` elements inside of ``job_configs``.

Allowing DataSources to be referenced in ``job_configs`` is an incremental
improvement that gives users the option of easily using DataSource objects in
the above cases to specify IO.


Proposed change
===============

Add optional boolean config values on a JobExecution that cause Sahara to
to treat values in ``job_configs`` as potential names or uuids of DataSource
objects. This applies to configuration values, parameters, and arguments
for all job types for maximum flexibility -- Hive and Pig jobs use parameters
to pass values, MapReduce uses configuration values, and Java and Spark use
arguments.

If Sahara resolves a value to the name or uuid of a DataSource it will
substitute the path information from the DataSource for the value and update
the job configuration as necessary to support authentication. If a value does
not resolve to a DataSource name or uuid value, the original value will be
used.

Note that the substitution will occur during submission of the job to the
cluster but will *not* alter the original JobExecution. This means that if
a user relaunches a JobExecution or examines it, the original values will be
present.

The following non mutually exclusive configuration values will control this
feature:

* edp.substitue_data_source_for_name (bool, default False)

  Any value in the ``args`` list, ``configs`` dict, or ``params`` dict in
  ``job_configs`` may be the name of a DataSource object.

* edp.substitute_data_source_for_uuid (bool, default False)

  Any value in the ``args`` list, ``configs`` dict or ``params`` dict in
  ``job_configs`` may be the uuid of a DataSource object.

This change will be usable from all interfaces: client, CLI, and UI. The UI may
choose to wrap the feature in some way but it is not required.  A user could
simply specify the config options and the DataSource identifiers on the job
execution configuration panel.


Alternatives
------------

A slightly different approach could be taken in which DataSource names or uuids
are prepended with a prefix to identify them. This would eliminate the need for
config values to turn the feature on and would allow individual values to be
looked up rather than all values. It would be unambiguous but may hurt
readability or be unclear to new users.

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

None required.  However, I can imagine that the UI might benefit from some
simple tooling around this feature, like checkboxes to enable the feature
on a Spark or Java job submission panel.


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  <tmckay>

Other contributors:
  <croberts>

Work Items
----------

* Support in Sahara
* Document
* Support in the UI (optional, it will work without additional work)

Dependencies
============


Testing
=======

Unit tests

Documentation Impact
====================

We will need to document this in the sections covering submission of jobs
to Sahara


References
==========
