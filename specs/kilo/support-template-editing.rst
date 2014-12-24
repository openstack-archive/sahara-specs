..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Spec - Add support for editing templates
==========================================

Blueprints:

Sahara Service: https://blueprints.launchpad.net/sahara/+spec/support-template-editing

Horizon UI:  https://blueprints.launchpad.net/horizon/+spec/data-processing-edit-templates

Currently, once a node group template or a cluster template has been
created, the only operations available are "copy" or "delete".  That has
unfortunate consequences on a user's workflow when they want to change a
template.  For example, in order to make even a small change to a node group
template that is used by a cluster template, the user must make a copy of
the node group template, change it, save it, and then create a new (or copy)
cluster template that uses the new node group template.  Ideally,
a user could just edit the template in place and move on.

Problem description
===================

Sahara currently lacks implementation for the update operation for both node
group and cluster templates.  In order to provide an implementation for
these operations, the following issues must be addressed.

1. What to do with existing clusters that have been built by using a
template that is being edited.  Would we automatically scale a cluster if a
cluster template was changed in a way that added/removed nodes?  Would we
shut down or start up processes if a node group template changed to
add/remove processes?

I propose that for this iteration, a user may not change any templates that
were used to build a currently running cluster.  This is consistent with the
rule currently in place to not allow deletion of templates that were are
used by a currently active cluster.  A future iteration could remove that
restriction for both delete and edit.
A user could still make a copy of a template and edit that if they wanted to
start a new cluster with the altered version of the template.

2. Make sure that all cluster templates that are dependant upon an edited
node group template will pick-up the changes to the node group template.

This is only a problem if we go the route of allowing templates used by an
active cluster to be edited.  In that case, we would need to change the
existing templates to reference the newly created template.


Proposed change
===============

The put (update) method will be implemented for both node group and cluster
templates.  Currently, the stubs are there, but they just return "Not
Implemented".

I think that the simplest method of sending an updated template is to send
the entire new template rather than just a diff.  It could be slightly
inefficient to do it this way, but avoids the need to build-in the diff
logic in the UI.  The current client library methods for update() seem to be
anticipating that sort of implementation.  If we went to sending a diff,
we may need to adjust the client library.

Alternatives
------------

Updates could be achieved by doing a delete/add combination,
but that is not a very good solution to this problem since any given node
group or cluster template could be referenced by another object.  Deleting
and adding would require updating each referencing object.

If we need to be able to edit templates that are used by an active cluster,
the edited version of the template will retain the ID and name of the
original template.  Prior to overwriting the original template,
it will be saved with a new ID and some form of "dotted" name to
indicate the version ("origname.1").  All running clusters would be changed
to reference the original template with the new ID.

Data model impact
-----------------

N/A

REST API impact
---------------

N/A.
The update operation are already defined in the API, but they are not yet
implemented.

Other end user impact
---------------------

The Horizon UI will be updated to include "Edit" as an operation for each
node group and cluster template.  The edit button will appear in the list of
operations in each row of the table.

The python-saharaclient library already defines the update() methods that
will be used for implementing this feature in the Horizon UI.  No changes to
python-saharaclient are anticipated to support this feature.

Deployer impact
---------------

N/A

Developer impact
----------------

N/A

Sahara-image-elements impact
----------------------------

N/A

Horizon impact
--------------

Horizon will be updated to include an "edit" button in each row of both the
node group and cluster templates tables.  That edit button will bring up the
edit form (essentially the "create" form, but with all the values
pre-populated from the existing template).  Clicking on "Save" from the edit
form will result in a call to the node group/cluster template update()
method in the python-saharaclient library.


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  croberts

Other contributors:
  tmckay

Work Items
----------

Horizon UI: croberts
Sahara service: tmckay


Dependencies
============

N/A

Testing
=======

There will be unit and integration tests added in Sahara.

Horizon will have a test added to the node group and cluster template panels
to verify that the appropriate forms are generated when edit is chosen.

Documentation Impact
====================

The Sahara UI user guide should be updated to note the availability of edit
functionality for templates.

References
==========

N/A
