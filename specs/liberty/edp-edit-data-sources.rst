..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

======================================
[EDP] Allow editing datasource objects
======================================

https://blueprints.launchpad.net/sahara/+spec/edp-edit-data-sources

Currently there is no way to edit a datasource object. If a path needs
to be changed, for example, the datasource must be deleted and a new
one created. The most common use case is a situation where a user
creates a datasource, runs a job, and receives an error from the job
because the path does not exist. Although it is not strictly necessary,
editable datasource objects would be a convenience when a user needs to
correct a path or credentials.

Problem description
===================

There are no API methods for updating a datasource object in the
REST API or at the conductor level.

The only way to correct a datasource is to delete an existing one and
create a new one with corrected information. Although it is possible to
use the same name, the object id will be different.

If editing is allowed, a user only needs to do a single operation to
make corrections. Additionally, the id is preserved so that objects which
reference it will reference the corrected path.

In the general case, editing a datasource should not be a problem for
a job execution which references it. Once a job execution enters the "RUNNING"
state, any information in datasource objects it references has been extracted
and passed to the process running the job. Consequently, editing a datasource
referenced by running or completed jobs will cause no errors. On relaunch,
a job execution will extract the current information from the datasource.

There is only a small window where perhaps editing should not be allowed.
This is when a datasource object is referenced by a job execution in the
"PENDING" state. At this point, information has not yet been extracted
from the datasource object, and a change during this window would
cause the job to run with paths other than the ones that existed at submission
time.

Proposed change
===============

Add an update operation to the REST API for datasource objects. Do not
allow updates for datasource objects that are referenced by job executions
in the "PENDING" state (this can be checked during validation).

Datasource objects referenced by job executions that are not in the PENDING
state may be changed. In an existing blueprint and related CR (listed in the
reference section) the URLs used by a job execution will be recorded in
the job execution when the job enters the RUNNING state. This means that for
any running or completed job execution, the list of exact datasource URLs
used in the execution will be available from the job execution itself even
if the referenced datasource has been edited.

Allow any fields in a datasource object to be updated except for id.
The object id should be preserved.

Add the corresponding update operation to the python-saharaclient.

Alternatives
------------

Do nothing

Data model impact
-----------------

None

REST API impact
---------------

Backward compatiblity will be maintained since this is a new endpoint.

**PUT /v1.1/{tenant_id}/data-sources/{data_source_id}**

Normal Response Code: 202 (ACCEPTED)

Errors: 400 (BAD REQUEST), 404 (NOT FOUND)

Update the indicated datasource object

**Example**
    **request**

    .. sourcecode:: http

        PUT http://sahara/v1.1/{tenant_id}/data-sources/{data_source_id}

    .. sourcecode:: http

        {
            "description": "some description",
            "name": "my_input",
            "url": "swift://container/correct_path"
        }

    **response**

    .. sourcecode:: http

        HTTP/1.1 202 ACCEPTED
        Content-Type: application/json

    .. sourcecode:: json

        {
            "created_at": "2015-04-08 20:27:13",
            "description": "some_description",
            "id": "7b25fc64-5913-4bc3-aaf4-f82ad03ea2bc",
            "name": "my_input",
            "tenant_id": "33724d3bf3114ae9b8ab1c170e22926f",
            "type": "swift",
            "updated_at": "2015-04-09 10:27:13",
            "url": "swift://container_correct_path"
        }


Other end user impact
---------------------

This operation should be added to the python-saharaclient API as well

$ sahara data-source-update [--name NAME] [--id ID] [--json]

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

To take advantage of this from the Horizon UI, we would need a selectable
"Edit" action for each datasource on the datasources page

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  Trevor McKay

Other contributors:
  Chad Roberts

Work Items
----------

Add REST and support methods to Sahara
Add operation to python-saharaclient
Add operation to datasource screens in Horizon
Add to WADL in api-ref

Dependencies
============

None


Testing
=======

Unit tests in Sahara and python-saharaclient

Documentation Impact
====================

Potentially any user documentation that talks about relaunch, or
editing of other objects like templates

References
==========

https://blueprints.launchpad.net/sahara/+spec/edp-datasource-placeholders
https://review.openstack.org/#/c/158909/
