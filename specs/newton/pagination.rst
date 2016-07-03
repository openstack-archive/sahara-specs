..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=====================================================
Adding pagination and sorting ability to Sahara
=====================================================

https://blueprints.launchpad.net/sahara/+spec/pagination

This specification describes an option for pagination ability which will admit
to work with objects in Sahara with API, CLI and Dashboard.
Also this specification describes sorting ability to Sahara API.

Problem description
===================


We are working on adding the ability for pagination to work with objects.
But having too many objects to be shown is not very user friendly. We want
to implement pagination so the dashboard can split the list of pages making
the UI better. User has ability to sort objects by columns in dashboard, but
if we add pagination ability, we will need to add sorting ability in API,
because ordering on UI side only will cause inconvenience since the service
will continue returning pages ordered by the default column.

Proposed change
===============

Will be added four optional parameters in API GET requests, which return
a lists of objects.

.. note:: Now elements on lists sorted by date of creation.

``marker`` - index of the last element on the list, which won't be in response.

``limit`` - maximum count of elements in response. This argument must be
positive integer number. If this parameter isn't passed, in response will be
all the elements which follow the element with id in marker parameter.
Also if ``marker`` parameter isn't passed, response will contain first
objects of the list with count equal ``limit``. If both parameters aren't
passed, API will work as usual.

``sort_by`` - name of the field of object which will be used by sorting.
If this parameter is passed, objects will be sorting by date of creation,
otherwise by this field.

The field can possess one of the next values for every object:

For Node Group Template:
``name``, ``plugin``, ``version``, ``created_at``, ``updated_at``


For Cluster Templates:
``name``, ``plugin``, ``version``, ``created_at``, ``updated_at``


For Clusters:
``name``, ``plugin``, ``version``, ``status``, ``instance_count``


For Job Binaries and Job Binaries Internal
``name``, ``create``, ``update``


For Data Sources:
``name``, ``type``, ``create``, ``update``


For Job Templates:
``name``, ``type``, ``create``, ``update``


For Jobs:
``id``, ``job_template``, ``cluster``, ``status``, ``duration``


By default Sahara api will return list in ascending order. Also if the user
wants a descending order list, he can use ``-`` prefix for ``sort_by``
argument.


Examples:

Get list of jobs in ascending order sorted by name.

**request**
``GET http://sahara/v1.1/775181/jobs?sort_by=name``

Get list of jobs in descending order sorted by name.

**request**
``GET http://sahara/v1.1/775181/jobs?sort_by=-name``


For convenience, collections contain atom "next" and "previous" markers.
The first page on the list doesn't contain a previous marker, the last page
on the list doesn't contain a next link. The following examples illustrate
pages in a collection of cluster templates.


Example:

Get one cluster template after template with ``id=3``.

**request**

``GET http://sahara/v1.0/775181/cluster-templates?limit=1&marker=3``

**response**

.. sourcecode:: json

    {
        "cluster_templates": [
            {
                "name": "cluster-template",
    "plugin_name": "vanilla",
                "id": "4",
                "node_groups": [
                    {
                        "name": "master",
                    },
                    {
                        "name": "worker",
                    }
                ],
            }
        ],
        "markers":
            {
                "next": "32",
                "previous": "22"
            }
    }


Example:
Let cluster template with id = 5 will be the last of collection.
Response will contain only "previous" link.

**request**

``GET http://sahara/v1.0/775181/cluster-templates?limit=1&marker=4``

**response**

.. sourcecode:: json

   {
      "cluster_templates":[
         {
            "description":"",
            "node_groups":[
               {
                  "name":"master",
               },
               {
                  "name":"worker",
               }
            ],
            "name":"cluster-template-2",
            "id":"5",
         }
      ],
      "markers":
          {
              "previous": "3"
          }
   }

Alternatives
------------

None

Data model impact
-----------------

None

REST API impact
---------------

Add ability get ``marker``, ``limit``,  ``sort_by`` parameters
in next requests:

Sahara API v1.0
~~~~~~~~~~~~~~~~~~~~~

``GET /v1.0/{tenant_id}/images``

``GET /v1.0/{tenant_id}/node-group-templates``

``GET /v1.0/{tenant_id}/cluster-templates``

``GET /v1.0/{tenant_id}/clusters``

Sahara API v1.1
~~~~~~~~~~~~~~~~~~~~~

``GET /v1.1/{tenant_id}/data-sources``

``GET /v1.1/{tenant_id}/job-binary-internals``

``GET /v1.1/{tenant_id}/job-binaries``

``GET /v1.1/{tenant_id}/jobs``

``GET /v1.1/{tenant_id}/job-executions``

Sahara API v2
~~~~~~~~~~~~~~~~~~~~~

``GET /v2/cluster-templates``

``GET /v2/clusters``

``GET /v2/data_sources``

``GET /v2/images``

``GET /v2/job-binaries``

``GET /v2/jobs``

``GET /v2/job-templates``

``GET /v2/node-group-templates``


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

Pagination will be added to Sahara-Dashboard via Horizon abilities. Now we
are using ``DataTable`` class to represent lists of data objects.
This class supports pagination abilities.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  mlelyakin (mlelyakin@mirantis.com)

Work Items
----------

* Adding ability of take arguments ``marker``, ``limit`` in Sahara API
* Adding unit tests for new features.
* Adding ability of take argument ``sort_by`` in Sahara API
* Adding this abilities in Sahara CLI Client
* Adding this abilities in Dashboard
* Documented pagination and sorting features

Dependencies
============

None

Testing
=======

Will be covered with unit tests.

Documentation Impact
====================

Will be adding to documentation of Sahara API.

References
==========

None
