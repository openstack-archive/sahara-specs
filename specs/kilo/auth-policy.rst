..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

============================
Authorization Policy Support
============================

Openstack components are supposed to check user privileges for performed
actions. Usually these checks are role-based. See
http://docs.openstack.org/developer/keystone/architecture.html#approach-to-authorization-policy.
Sahara need to support policies too.

Problem description
===================

OpenStack administrators may want to tune authorization policy for Sahara.
There should be a way to restrict some users to perform some of Sahara
operations.

Proposed change
===============

Add auth check for all Sahara API endpoints. This could be done in the same
way as in other openstack component. There is "policy" module in Oslo library
that may do all underlying work.

Proposed content of the policy file:

.. sourcecode:: json

    {
        "context_is_admin": "role:admin",
        "admin_or_owner":  "is_admin:True or project_id:%(project_id)s",
        "default": "rule:admin_or_owner",

        "clusters:get_all": "",
        "clusters:create": "",
        "clusters:scale": "",
        "clusters:get": "",
        "clusters:delete": "",

        "cluster-templates:get_all": "",
        "cluster-templates:create": "",
        "cluster-templates:get": "",
        "cluster-templates:modify": "",
        "cluster-templates:delete": "",

        "node-group-templates:get_all": "",
        "node-group-templates:create": "",
        "node-group-templates:get": "",
        "node-group-templates:modify": "",
        "node-group-templates:delete": "",

        "plugins:get_all": "",
        "plugins:get": "",
        "plugins:get_version": "",
        "plugins:convert_config": "",

        "images:get_all": "",
        "images:get": "",
        "images:register": "",
        "images:unregister": "",
        "images:add_tags": "",
        "images:remove_tags": "",

        "job-executions:get_all": "",
        "job-executions:get": "",
        "job-executions:refresh_status": "",
        "job-executions:cancel": "",
        "job-executions:delete": "",

        "data-sources:get_all": "",
        "data-sources:get": "",
        "data-sources:register": "",
        "data-sources:delete": "",

        "jobs:get_all": "",
        "jobs:create": "",
        "jobs:get": "",
        "jobs:delete": "",
        "jobs:get_config_hints": "",
        "jobs:execute": "",

        "job-binaries:get_all": "",
        "job-binaries:create": "",
        "job-binaries:get": "",
        "job-binaries:delete": "",
        "job-binaries:get_data": "",

        "job-binary-internals:get_all": "",
        "job-binary-internals:create": "",
        "job-binary-internals:get": "",
        "job-binary-internals:delete": "",
        "job-binary-internals:get_data": ""
    }

Separating Sahara users and operators could be the next step.

Alternatives
------------

None.

Data model impact
-----------------

None.

REST API impact
---------------

None.

Other end user impact
---------------------

None.

Deployer impact
---------------

None.

Developer impact
----------------

Adding new API will require changing policy rules.

Sahara-image-elements impact
----------------------------

None.

Sahara-dashboard / Horizon impact
---------------------------------

None.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  alazarev (Andrew Lazarev)

Work Items
----------

* Add policy.py from oslo
* Add config options to control policy file and settings
* Add policy check to all API calls
* Add unit tests
* Add documentation

Dependencies
============

* Policy module in Oslo.

Testing
=======

* Unit tests
* Manual testing

Documentation Impact
====================

* Feature need to be documented

References
==========

* http://docs.openstack.org/developer/keystone/architecture.html#approach-to-authorization-policy
* http://docs.openstack.org/developer/keystone/api/keystone.openstack.common.policy.html
* http://docs.openstack.org/developer/keystone/configuration.html#keystone-api-protection-with-role-based-access-control-rbac
