..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

====================================
Objects update support in Sahara API
====================================

https://blueprints.launchpad.net/sahara/+spec/api-for-objects-update

This specification proposes to add api calls that allows to update objects that
currently cant be updated this way.

Problem description
===================

Current Sahara API doesn't support update of some objects, which can be
required by some other features (for example it's needed for shared and
protected resources implementation that will be proposed in ACL spec later).

Updates are already implemented for node group templates, cluster templates,
job binaries, data sources and should be done for clusters,
jobs, job executions and job binary internals.

Proposed change
===============

Update operation will be added for cluster, job, job execution and job binary
internal objects.

For clusters and jobs only description and name update will be allowed for now.
For job binary internals only name will be allowed to update.
There is nothing will be allowed to update for job executions, only
corresponding methods will be added.

Also will be added support of PATCH HTTP method to modify existing resources.
It will be implemented the same way as current PUT method.

Alternatives
------------

None

Data model impact
-----------------

None

REST API impact
---------------
Following API calls will be added:

**PATCH /v1.1/{tenant_id}/clusters/{cluster_id}**

**PATCH /v1.1/{tenant_id}/jobs/{job_id}**

**PATCH /v1.1/{tenant_id}/job-executions/{job_execution_id}**

**PATCH /v1.1/{tenant_id}/job-binary-internals/{job_binary_internal_id}**

Other end user impact
---------------------

This update methods will be added to saharaclient API.

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

This update methods will not be added to Horizon yet, but will be added later
as part of ACL spec.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  apavlov-n

Work Items
----------

* Adding PATCH method;
* Adding new API calls;
* Adding operations to saharaclient;
* Documentation update in api-ref.

Dependencies
============

None

Testing
=======

Unit tests and API tests in tempest will be added.

Documentation Impact
====================

Sahara REST API documentation in api-ref will be updated.

References
==========

None
