..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

================================================================
upgrade oozie Web Service API version of sahara edp oozie engine
================================================================

https://blueprints.launchpad.net/sahara/+spec/update-oozie-client-version

This spec is to upgrade oozie web service api version of oozie engine.

Problem description
====================

Currently sahara oozie server version is 4.1.0, but oozie engine still
use v1 oozie web service api which is used in oozie 3.x. By upgrading
oozie web service api from v1 to v2, we can add more oozie features
into sahara.

Proposed change
===============

change sahara OozieClient job_url and jobs_url from /v1/job/%s,
/v1/jobs to /v2/job/%s, /v2/jobs.

for /v2/jobs, remains the same as /v1/jobs

for /v2/job/, there is a difference in the JSON format of job
information API,particularly for map-reduce action,no changes
for other actions.In v1, externalId and consoleUrl point to
spawned child job ID, and exteranlChildIDs is null in map-reduce
action. In v2, externalId and consoleUrl point to launcher job ID,
and exteranlChildIDs is spawned child job ID in map-reduce action.
this exteranlChildIDs can be used for recurrence edp job's child
job id.

here are the new oozie features can be added into sahara.

(1)PUT oozie/v2/job/oozie-job-id?action=update, we can update
job's definition and properties.
(2)GET /oozie/v2/job/oozie-job-id?show=errorlog, we can get oozie
error log when job is failed, so we can show user the detail error
information. Currently sahara edp engine tells user nothing when
job is failed.

so we can add update_job() and show_error_log() into oozie client.
details about these two features will be drafted in another spec.

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

None

Developer impact
----------------

None

Sahara-image-elements impact
----------------------------

None

Sahara-dashboard / Horizon impact
---------------------------------


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

* update oozie client in oozie engine

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

oozie web service api
http://oozie.apache.org/docs/4.2.0/WebServicesAPI.html
