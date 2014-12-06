..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

========================================================
[EDP] Add a Spark job type (instead of overloading Java)
========================================================

https://blueprints.launchpad.net/sahara/+spec/edp-spark-job-type

Spark EDP has been implemented initially using the Java job type. However,
the spark semantics are slightly different and Spark jobs will probably
continue to diverge from Java jobs during future development.  Additionally,
a specific job type will help users distinguish between Spark apps and Java
mapreduce jobs in the Sahara database.


Problem description
===================

The work involves adding a Spark job type to the job type enumeration in
Sahara and extending the dashboard to allow the creation and submission
of Spark jobs. The Sahara client must be able to create and submit Spark
jobs as well (there may not be any new work in the client to support this).

Existing unit tests and integration tests must be repaired if the addition
of a new job type causes them to fail. Unit tests analagous to the tests
for current job types should be added for the Spark job type.

Integration tests for Spark clusters/jobs will be added as a separate effort.

Proposed change
===============

Changes in the Sahara-api code:

* Add the Spark job type to the enumeration
* Add validation methods for job creation and job execution creation
* Add unit tests for the Spark job type
* Add the Spark job type to the job types supported by the Spark plugin

  + Leave the Java type supported for Spark until the dashboard is changed

* Add config hints for the Spark type

  + These may be empty initially

Changes in the Sahara dashboard:

* Add the Spark job type as a selectable type on the job creation form.

  + Include the "Choose a main binary" input on the Create Job tab
  + Supporting libraries are optional, so the form should include the Libs tab

* Add a "Launch job" form for the Spark job type

  + The form should include the "Main class" input.
  + No data sources, as with Java jobs
  + Spark jobs will share the edp.java.main_class configuration with Java jobs.
    Alternatively, Spark jobs could use a edp.spark.main_class config
  + There may be additional configuration parameters in the future, but none
    are supported at present.  The Configuration button may be included or
    left out.
  + The arguments button should be present

Alternatives
------------

Overload the existing Java job type.  It is similar enough to work as a
proof of concept, but long term this is probably not clear, desirable or
maintainable.

Data model impact
-----------------

None. Job type is stored as a string in the database, so there should be
no impact there.

REST API impact
---------------

None. The JSON schema will list "Spark" as a valid job type, but the API
calls themselves should not be affected.

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

Described under Proposed Change.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  Trevor McKay (sahara-api)

Other contributors:
  Chad Roberts (dashboard)

Work Items
----------

Additional notes on implementation of items described under Proposed Change:

* The simple addition of JOB_TYPE_SPARK to sahara.utils.edp.JOB_TYPES_ALL
  did not cause existing unit tests to fail in an experiment

* Existing unit tests should be surveyed and analagous tests for the Spark job
  type should be added as appropriate

* sahara.service.edp.job_manager.get_job_config_hints(job_type) needs to handle
  the Spark job type. Currently all config hints are retrieved from the Oozie
  job engine, this will not be correct.

  + Related, the use of JOB_TYPES_ALL should probably be modified in
    workflow_creator.workflow_factory.get_possible_job_config() just to be safe

* sahara.service.edp.job_utils.get_data_sources() needs to treat Spark jobs
  like Java jobs (there are no data sources, only arguments)

* service.validations.edp.job.check_main_libs() needs to require a main
  application jar for Spark jobs and allow supporting libs as optional

* service.validations.edp.job_executor.check_job_executor()

  + Spark requires edp.java.main_class (or edp.spark.main_class)
  + check_edp_job_support() is called here and should be fine. The default is
    an empty body and the Spark plugin does not override this method since the
    Spark standalone deployment is part of a Spark image generated from DIB

* Use of EDP job types in sahara.service.edp.oozie.workflow_creator should not
  be impacted since Spark jobs shouldn't be targeted to an Oozie engine by the
  job manager (but see note on get_job_config_hints() and JOB_TYPES_ALL)

* The Sahara client does not appear to reference specific job type values so
  there is likely no work to do in the client

Dependencies
============

This depends on https://blueprints.launchpad.net/sahara/+spec/edp-spark-standalone


Testing
=======

New unit tests will be added for the Spark job type, analogous to existing
tests for other job types. Existing unit and integration tests will ensure that
other job types have not been broken by the addition of a Spark type.


Integration tests for Spark clusters should be added in the following
blueprint, including tests for EDP with Spark job types

https://blueprints.launchpad.net/sahara/+spec/edp-spark-integration-tests

Documentation Impact
====================

The User Guide calls out details of the different job types for EDP.
Details of the Spark type will need to be added to this section.


References
==========

None.
