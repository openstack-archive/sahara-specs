..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================================
Add ability of suspending and resuming EDP jobs for sahara
==========================================================

https://blueprints.launchpad.net/sahara/+spec/
add-suspend-resume-ability-for-edp-jobs

This spec is to allow suspending and resuming edp jobs in sahara.

Problem description
====================

Currently sahara does not allow suspending and resuming edp jobs.
But in some use cases, for Example, one edp job containing many steps,
after finishing one step, user want to suspend this job and check the
output, and then resume this job. So by adding suspending and resuming
ability to sahara edp engine, we can have different implementation for
different engine.(oozie,spark,storm etc)

Proposed change
===============

Add one api interface in sahara v11 API.

Define suspend_job() and resume_job() interface in sahara base edp engine,
then implement this interface to the oozie engine. (Spark and storm engine
will be drafted in later spec)

Add "SUSPENDED" and "PREPSUSPENDEDED" in the sahara. only the job's status
is "RUNNING" or "PREP" can we suspend this job. and make the job's status
shown as "SUSPENDED" or "PREPSUSPENDED".

Add a validation dict named suspend_resume_supported_job_type = {} to check
which job type is allowed to suspend and resume when request comes in.

If the job's status is not in RUNNING or PREP, for example, job is already
finished, we do validation check, and there is no suspend or resume action.

Example of suspending or resuming an edp job

PATCH /v1.1/{tenant_id}/job-executions/<job_execution_id>

.. sourcecode::json

    {
        "info": {
            "status": "suspend" or "resume"
        }
    }

response:

HTTP/1.1 202 Accepted
Content-Type: application/json


Add one api interface in the python-sahara-client

For oozie implementation, we just call oozie client to invoke suspend and
resume API.

For spark and storm implementation, there is no implementation now, and we
will add them later.

Alternatives
------------

None

Data model impact
-----------------

None

REST API impact
---------------

Add one API interface.
PATCH /v1.1/{tenant_id}/job-executions/<job_execution_id>

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

add two combobox item named "suspend job" and "resume job" option
at the right side of the Job list table.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
   luhuichun(lu huichun)

Other contributors:
  None

Work Items
----------

* add one api in v11
* add suspend_job and resume_job in base engine and oozie engine
* add two new job status "SUSPENDED" and "PREPSUSPENDED".
* add two api interface in python-sahara-client
* modify sahara api reference docs
* Add task to update the WADL at api-site

Dependencies
============

None.

Testing
=======

unit test in edp engine
add scenario integration test

Documentation Impact
====================

Need to be documented.

References
==========

oozie suspend and resume jobs implementation
https://oozie.apache.org/docs/4.0.0/CoordinatorFunctionalSpec.html
