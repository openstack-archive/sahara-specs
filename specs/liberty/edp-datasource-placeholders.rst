..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=====================================
Allow placeholders in datasource URLs
=====================================

https://blueprints.launchpad.net/sahara/+spec/edp-datasource-placeholders

This spec is to allow using placeholders in EDP data source URL.

Problem description
===================

Common use case: user wants to run EDP job two times. Now the only way to do
that with the same data sources is to erase result of the first run before
running job the second time. Allowing to have random part in URL will allow
to use output with random suffix.

Proposed change
===============

Introduce special strings that could be used in EDP data source URL and will
be replaced with appropriate value.

The proposed syntax for placeholder is %FUNC(ARGS)%.

As a first step I suggest to implement two functions only:

* %RANDSTR(len)% - will be replaced with random string of lowercase letters of
  length ``len``.
* %JOB_EXEC_ID% - will be replaced with the job execution ID.

Placeholders will not be allowed in protocol prefix. So, there will be no
validation impact.

List of functions could be extended later (e.g. to have %JOB_ID%, etc.).

URLs after placeholders replacing will be stored in ``job_execution.info``
field during job_execution creation. This will allow to use them later to find
objects created by a particular job run.

Example of create request for data source with placeholder:

.. sourcecode:: json

    {
        "name": "demo-pig-output",
        "description": "A data source for Pig output, stored in Swift",
        "type": "swift",
        "url": "swift://edp-examples.sahara/pig-job/data/output.%JOB_EXEC_ID%",
        "credentials": {
            "user": "demo",
            "password": "password"
        }
    }


Alternatives
------------

Do not allow placeholders.

Data model impact
-----------------

``job_execution.info`` field (json dict) will also store constructed URLs.

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

Horizon need to be updated to display actual URLs for job execution. Input
Data Source and Output Data Source sections of job execution details page will
be extended to include information about URLs used.

REST will not be changed since new information is stored in the existing
'info' field.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  alazarev (Andrew Lazarev)

Other contributors:
  None

Work Items
----------

* Implement feature
* Document feature

Dependencies
============

None.

Testing
=======

Manually.

Documentation Impact
====================

Need to be documented.

References
==========

None