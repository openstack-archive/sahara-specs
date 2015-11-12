..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=====================================================================
Modify 'is_default' behavior relative to 'is_protected' for templates
=====================================================================

https://blueprints.launchpad.net/sahara/+spec/replace-is-default

With the addition of the general *is_protected* field on all objects,
the *is_default* field on cluster and node group templates is semi-redundant.
Its semantics should be changed; specifically, gating of update and
delete operations should be handled exclusively by *is_protected*. The
*is_default* field should still be used to identify default templates for bulk
update and delete operations by the sahara-templates tool.


Problem description
===================

The *is_protected* and *is_default* fields both control gating of
update and delete operations. This overlap is not clean and should
be resolved to avoid confusion for end users and developers alike
and to decrease complexity in code maintenance.

Gating of update or delete operations should no longer be associated
with the *is_default* field.  Instead, it should only mark the default
templates created with the *sahara-templates* tool so that bulk update
and delete operations continue to function as is.

Proposed change
===============

Remove the checks in the sahara db layer that raise exceptions based
on the *is_default* field for template update and delete operations. This
will leave the gating under the control of *is_protected*. Note that
there will no longer be an error message that informs the user of
failure to update or delete a default template -- it will be phrased in
terms of the *is_protected* field instead.

Modify all of the default template sets bundled with sahara to include
the *is_protected* field set to *True*. This will ensure that templates
used as-is from the sahara distribution will be created with update and
delete operations hindered.

Provide a database migration that sets *is_protected* to **True** for
all templates with *is_default* set to **True**. This will guarantee
the hindrance on update and delete operations continues for existing default
templates.

Currently, sahara will allow update and delete of default templates if the
*ignore_defaults* flag is set on the conductor methods. Change the name
and handling of this flag slightly so that if it is set, sahara will ignore
the *is_protected* flag instead.  This will allow the *sahara-templates*
tool to easily update and delete default templates without having to first
clear the *is_protected* flag.

Write a documentation section on default templates best practices. Given the
addition of *is_protected* and *is_public* and the changed semantics for
*is_default*, provide some suggestions for best practices as follows: operators
will be encouraged to load the default template sets in the *admin* or
*service* tenant and then make them available to users with *is_public*. If
modifications will be made to the default templates, they can be loaded, copied
and modified and the resultant templates made available to users with
*is_public*. This will ensure that the original default templates as bundled
with sahara will always be available as a reference with *is_protected* set to
*True*. Alternatively, an admin could copy the default templates directory from
the sahara distribution to another location and modify them on disk before
loading them.

Implications of the change: it will now be possible for tenants with default
templates to set *is_protected* to *True* and then edit those templates. If
a subsequent update of those templates is done from the *sahara-templates*
tool, those edits will be overwritten. This should be made clear in the best
practices documentation described above. Additionally, *is_default* will remain
a field that is only settable from the *sahara-templates* tool so that the
default sets can be managed as a group.



Alternatives
------------

None. Goals of the original default templates implementation were:

1 allow offline update and delete of default templates
2 treat a group of node group and cluster templates as a coherent set
3 allow operation filtered by plugin and version

The first requirement precludes use of the sahara client to manage default
templates. If we did not have the first requirement, we could conceivably
use the sahara client to manage them but requirements two and three
mean that we would need logic around the operations anyway. Lastly,
removing the *is_default* field completely leaves us with a need for
some other mechanism to identify default templates. It seems best to
remove the overlapping functionality of *is_default* and *is_protected* but
leave the rest of the current implementation unchanged.

Also, this could be deferred as part of the API V2 initiative, but since it has
so little impact on the API (see below) it seems unnecessary to wait

Data model impact
-----------------

None, but provide an appropriate migration to set *is_protected* to
**True** for all existing templates with *is_default* set to **True**.

REST API impact
---------------

None, *is_default* is only accessible directly through the conductor layer.

Other end user impact
---------------------

None

Deployer impact
---------------

Deployers should be familiar with the best practices documentation we will
add concerning default templates.

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

Primary assignee:
  tmckay

Other contributors:
  None

Work Items
----------

Remove code in the sahara db layer that rejects update or delete for defaults
Update default templates bundled with sahara to include *is_protected*
Add alembic migration for setting *is_protected* on existing defaults
Review documentation and add a best practices section

Dependencies
============

None

Testing
=======

Unit tests will be sufficient. Existing tests will be modified
to prove that default templates may be updated and edited after
this change, assuming *is_protected* is False. The existing
tests on *is_protected* cover normal update/delete control.


Documentation Impact
====================

Documentation on the sahara-templates tool may change slightly where it
describes how to update or delete a default template, and we will add a
best practices section as noted

References
==========

None
