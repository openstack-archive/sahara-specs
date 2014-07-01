..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==============================================================
[EDP] Refactor job manager to support multiple implementations
==============================================================


https://blueprints.launchpad.net/sahara/+spec/edp-refactor-job-manager

The job manager at the core of Sahara EDP (Elastic Data Processing) was
originally written to execute and monitor jobs via an Oozie server. However,
the job manager should allow for alternative EDP implementations which
can support new cluster types or new job execution environments.


Problem description
===================

To date, the provisioning plugins released with Sahara all support deployment
of an Oozie server.  Oozie was a logical choice for the early releases of
Sahara EDP because it provided commonality across several plugins and allowed
the rapid development of the EDP feature.

However, Oozie is built on top of Hadoop Mapreduce and not every cluster
configuration can or should support it (consider a Spark cluster, for example,
where Hadoop Mapreduce is not a necessary part of the install). As Sahara
supports additional cluster types and gains a wider install base, it's
important for EDP to have the flexibility to run on new cluster configurations
and new job execution paradigms.

The current implementation of the job_manager has hardcoded dependencies on
Oozie. These dependencies should be removed and the job manager should be
refactored to support the current Oozie implementation as well as new
implementations. The job manager should select the appropriate implementation
for EDP operations based on attributes of the cluster.


Proposed change
===============

Sahara EDP requires three basic operations on a job:

* Launch the job
* Poll job status
* Terminate the job

Currently these operations are implemented in job_manager.py with explicit
dependencies on the Oozie server and Oozie-style workflows.

To move these dependencies out of the job manager, we can define an abstract
class that contains the three operations::

    @six.add_metaclass(abc.ABCMeta)
    class JobEngine(object):
        @abc.abstractmethod
        def cancel_job(self, job_execution):
            pass

        @abc.abstractmethod
        def get_job_status(self, job_execution):
            pass

        @abc.abstractmethod
        def run_job(self, job_execution):
            pass

For each EDP implementation Sahara supports, a class should be derived from
JobEngine that contains the details.  Each implementation and its supporting
files should be contained in its own subdirectory.

Logic can be added to the job manager to allocate a JobEngine based on the
cluster and/or job type, and the job manager can call the appropriate method
on the allocated JobEngine.

As much as possible the JobEngine classes should be concerned only with
implementing the operations. They may read objects from the Sahara database as
needed but modifications to the job execution object should generally be
done in the job_manager.py (in some cases this may be difficult, or may require
optional abstract methods to be added to the JobEngine base class).

For example, the "cancel_job" sequence should look something like this:

1) "cancel_job(id)" is called in job_manager.py
2) The job manager retrieves the job execution object and the cluster
   object and allocates an appropriate job engine
3) The job manager calls engine.cancel_job(job_execution)
4) The engine performs necessary steps to cancel the job
5) The job manager traps and logs any exceptions
6) The job manager calls engine.get_job_status(job_execution)
7) The engine returns the status for the job, if possible
8) The job manager updates the job execution object in the Sahara database
   with the indicated status, if any, and returns

In this example, the job manager has no knowledge of operations in the engine,
only the high level logic to orchestrate the operation and update the status.


Alternatives
------------

It may be possible to support "true" plugins for EDP similar to the
implementation of the provisioning plugins. In this case, the plugins would be
discovered and loaded dynamically at runtime.

However, this requires much more work than a refactoring and introduction of
abstract classes and is probably not realistic for the Juno release. There are
several problems/questions that need to be solved:

* Should EDP interfaces simply be added as optional methods in the current
  provisioning plugin interface, or should EDP plugins be separate entities?

* Do vendors that supply the provisioning plugins also supply the EDP plugins?

* If separate EDP plugins are chosen, a convention is required to associate an
  EDP plugin with a provisioning plugin so that the proper EDP implementation
  can be chosen at runtime for a particular cluster without any internal glue

* For clusters that are running an Oozie server, should the Oozie EDP
  implementation always be the default if another implementation for the
  cluster is not specified? Or should there be an explicitly designated
  implementation for each cluster type?

* It requires not only a formal interface for the plugin, but a formal
  interface for elements in Sahara that the plugin may require.  For instance,
  an EDP plugin will likely need a formal interface to the conductor so that it
  can retrieve EDP objects (job execution objects, data sources, etc).


Data model impact
-----------------

This change should only cause one minor (optional) change to the current data
model. Currently, a JobExecution object contains a string field named
"oozie_job_id". While a job id field will almost certainly still be needed for
all implementations and can be probably be stored as a string, the name should
change to "job_id".

JobExecution objects also contain an "extra" field which in the case of the
Oozie implementation is used to store neutron connection info for the Oozie
server. Other implementations may need similar data, however since the "extra"
field is stored as a JsonDictType no change should be necessary.

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

Primary Assignee:
Trevor McKay


Work Items
----------




Dependencies
============

None

Testing
=======

Testing will be done primarily through the current unit and integration tests.
Tests may be added that test the selection of the job engine.


Documentation Impact
====================

None


References
==========

None
