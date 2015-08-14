..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=========================================
Support of shared and protected resources
=========================================

https://blueprints.launchpad.net/sahara/+spec/shared-protected-resources

This specification proposes to add ability of creation and modification of
shared across tenants and protected from updates objects.

Problem description
===================

Currently all objects created by Sahara are visible only from the tenant in
which they were created and not insured from occasional modification or
deletion.

Proposed change
===============

This specification proposes to add ``is_public`` and ``is_protected`` boolean
fields to all Sahara objects that can be accessed through REST API. They will
be added to clusters, cluster templates, node group templates, data sources,
job executions, jobs, job binaries and job binary internals.

All this objects can be created with enabled ``is_public`` and
``is_protected`` parameters which can be updated after creation with
corresponding API call. Both of them will be False by default.

If some object has ``is_public`` field set to ``True``, it means that it's
visible not only from the tenant in which it was created, but from any other
tenants too.

If some object has ``is_protected`` field set to ``True``, it means that it
could not be modified (updated, scaled, canceled or deleted) unless this field
will be set to ``False``. If ``is_protected`` parameter is set to ``True``,
object can be modified only if ``is_protected=False`` will be supplied in
update request.

Public objects created in one tenant can be used by other tenants (for example,
cluster can be created from public cluster template which is created in another
tenant), but to prevent management of resources in different tenants,
operations like update, delete, cancel and scale will be possible only from
tenant in which object was created.

To control this restrictions, a couple of methods will be implemented in
``sahara.service.validation.acl``:

.. sourcecode:: python

    def check_tenant_for_delete(context, object)
    def check_tenant_for_update(context, object)
    def check_protected_from_delete(object)
    def check_protected_from_update(object, data)

``check_tenant_for_*`` will compare tenant_id in context with object tenant_id
and if they different, raise an error. But this check should be skipped for
periodics as there is no tenant_id in context in this case.

``check_protected_from_delete`` will check ``is_protected`` field and if it's
set to True, raise an error.
``check_protected_from_update`` will additionally check that ``is_protected``
field wasn't changed to ``False`` with update data.

This methods will be called mostly in ``sahara.db.sqlalchemy.api`` inside of
update and delete methods that make only db changes. But for cluster_create,
cluster_scale, job_execute, job_execution_cancel and job_execution_delete
operations they will be called during validation before api calls.

Alternatives
------------

None

Data model impact
-----------------

Two extra fields ``is_public`` and ``is_protected`` will be added to
objects listed above.

REST API impact
---------------

New API calls will not be added, but existing ones will be updated to support
new fields.

Other end user impact
---------------------

Saharaclient API will be updated to support new fields.

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

``is_public`` and ``is_protected`` checkboxes will be added to ``Update`` and
``Create`` panels of each object.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  apavlov-n

Work Items
----------

* Adding new fields ``is_public`` and ``is_protected`` to objects listed above;
* Implementation of validations, described above;
* Updating saharaclient with corresponding changed;
* Documentation about new features will be added.

Dependencies
============

None

Testing
=======

Unit tests will be added and a lot of manual testing.

Documentation Impact
====================

All changes will be documented.

References
==========

None
