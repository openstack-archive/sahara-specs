..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=====================================================
[EDP] Add an engine for a Spark standalone deployment
=====================================================

https://blueprints.launchpad.net/sahara/+spec/edp-spark-standalone

The Spark provisioning plugin allows the creation of Spark standalone
clusters in Sahara.  Sahara EDP should support running Spark
applications on those clusters.


Problem description
===================

The Spark plugin uses the *standalone* deployment mode for Spark
(as opposed to Spark on YARN or Mesos). An EDP implementation must be created
that supports the basic EDP functions using the facilities provided by the
operating system and the Spark standalone deployment.

The basic EDP functions are:

* **run_job()**
* **get_job_status()**
* **cancel_job()**


Proposed change
===============

The Sahara job manager has recently been refactored to allow provisioning
plugins to select or provide an EDP engine based on the cluster and job_type.
The plugin may return a custom EDP job engine object or choose a default engine
provided by Sahara.

A default job engine for Spark standalone clusters can be added to Sahara
that implements the basic EDP functions.

Note, there are no public APIs in Spark for job status or cancellation
beyond facilities that might be available through a SparkContext
object instantiated in a Scala program. However, it is possible to
provide basic functionality without developing Scala programs.

* **Engine selection criteria**

  The Spark provisioning plugin must determine if the default Spark EDP
  engine may be used to run a particular job on a particular cluster. The
  following conditions must be true to use the engine

  + The default engine will use *spark-submit* for running jobs, therefore
    a cluster must have at least Spark version 1.0.0 to use the engine.

  + The job type must be Java (initially)

* **Remote commands via ssh**

  All operations should be implemented using ssh to run remote
  commands, as opposed to writing a custom agent and client. Furthermore,
  any long running commands should be run asynchronously.

* **run_job()**

  The run_job() function will be implemented using the *spark-submit*
  script provided by Spark.

  + *spark-submit* must be run using client deploy mode

  + The *spark-submit* command will be executed via ssh.  Ssh must
    return immediately and must return a process id (PID) that can be
    used for checking status or cancellation. This implies that the
    process is run in the background.

  + SIGINT must not be ignored by the process running *spark-submit*.
    Care needs to be taken here, since the default behavior of a
    process backgrounded from bash is to ignore SIGINT. (This can be
    handled by running *spark-submit* as a subprocess from a wrapper
    which first restores SIGINT, and launching the wrapper from ssh. In this
    case the wrapper must be sure to propagate SIGINT to the child).

  + *spark-submit* requires that the main application jar be in
    local storage on the node where it is run.  Supporting jars may
    be in local storage or hdfs. For simplicity, all jars will be uploaded
    to local storage.

  + *spark-submit* should be run from a subdirectory on the
    master node in a well-known location.  The subdirectory naming
    should incorporate the job name and job execution id from
    Sahara to make locating the directory easy. Program files,
    output, and logs should be written to this directory.

  + The exit status returned from *spark-submit* must be written
    to a file in the working directory.

  + *stderr* and *stdout* from *spark-submit* should be redirected
    and saved in the working directory. This will help debugging as well
    as preserve results for apps like SparkPi which write the result to stdout.

  + Sahara will allow the user to specify arguments to the Spark
    application. Any input and output data sources will be specified
    as path arguments to the Spark app.

* **get_job_status()**

  Job status can be determined by monitoring the PID returned
  by run_job() via *ps* and reading the file containing the exit status

  + The initial job status is PENDING (the same for all Sahara jobs)
  + If the PID is present, the job status is RUNNING
  + If the PID is absent, check the exit status file
    - If the exit status is 0, the job status is SUCCEEDED
    - If the exit status is -2 or 130, the job status is KILLED (by SIGINT)
    - For any other exist status, the job status is DONEWITHERROR

  + If the job fails in Sahara (ie, because of an exception), the job status
    will be FAILED

* **cancel_job()**

  A Spark application may be canceled by sending SIGINT to the
  process running *spark-submit*.

  + cancel_job() should send SIGINT to the PID returned by run_job().
    If the PID is the process id of a wrapper, the wrapper must
    ensure that SIGINT is propagated to the child

  + If the command to send SIGINT is successful (ie, *kill* returns 0),
    cancel_job() should call get_job_status() to update the job status


Alternatives
------------

The Ooyala job server is an alternative for implementing Spark EDP, but it's
a project of its own outside of OpenStack and introduces another dependency. It
would have to be installed by the Spark provisioning plugin, and Sahara
contributors would have to understand it thoroughly.

Other than Ooyala, there does not seem to be any existing client or API for
handling job submission, monitoring, and cancellation.

Data model impact
-----------------

There is no data model impact, but a few fields will be reused.

The *oozie_job_id* will store an id that allows the running application
to be operated on.  The name of this field should be generalized at some
point in the future.

The job_execution.extra field may be used to store additional information
necessary to allow operations on the running application


REST API impact
---------------

None

Other end user impact
---------------------

None.  Initially Spark jobs (jars) can be run using the Java job type.
At some point a specific Spark job type will be added (this will be
covered in a separate specification).


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

Trevor McKay


Work Items
----------

* implement default spark engine selection in spark provisioning plugin
* implement run
* implement get_job_status
* implement cancel
* implement launch wrapper
* implement unit tests

Dependencies
============

None


Testing
=======

Unit tests will be added for the changes in Sahara.

Integration tests for Spark standalone clusters will be added in another
blueprint and specification.


Documentation Impact
====================

The Elastic Data Processing section of the User Guide should talk about
the ability to run Spark jobs and any restrictions.


References
==========

