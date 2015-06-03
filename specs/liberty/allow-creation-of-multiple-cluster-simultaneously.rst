..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

======================================================
Allow the creation of multiple clusters simultaneously
======================================================

https://blueprints.launchpad.net/sahara/+spec/simultaneously-creating-multiple-clusters

We want to improve user experience when creating new clusters by allowing the
user to create multiple clusters at the same time.

Problem description
===================

When creating new clusters, the user has only the option to start a single
cluster. In some cases, for example when dealing with researches, the user
needs more than a single cluster with a given template. In order to reduce the
work of creating a cluster then going back to create a second one and so on, we
want to include the option of creating multiple clusters simultaneously by
adding an option of number of clusters.

Proposed change
===============

We want to introduce an option for the user to select how many clusters will be
spawned.
When creating multiple clusters we will add a sequential number to the given
cluster name (hadoop-cluster1, hadoop-cluster2, ...).

The creation workflow would go as follows:

* 1) The users will request the creation of multiple clusters
     POST v1.1/<project_id>/clusters/multiple using a body as described below.
* 2) The return of this call will be a list of clusters id
* 3) Finally the user will be able to track the cluster state using the ids.


Alternatives
------------

The user keeps creating a cluster at a time.

Data model impact
-----------------

None.

REST API impact
---------------

We need to create a new API call (create_multiple_clusters()) to allow the
creation of multiple clusters by passing a new parameter specifying the number
of clusters that will be created.

POST v1.1/<project_id>/clusters/multiple

Other end user impact
---------------------

We will also need to change the python-saharaclient to allow the creation of
multiple clusters.

* Request

{
    "plugin_name": "vanilla",
    "hadoop_version": "2.4.1",
    "cluster_template_id": "1beae95b-fd20-47c0-a745-5125dccbd560",
    "default_image_id": "be23ce84-68cb-490a-b50e-e4f3e340d5d7",
    "user_keypair_id": "doc-keypair",
    "name": "doc-cluster",
    "count": 2,
    "cluster_configs": {}

}

* Response

{clusters: ["c8c3fee5-075a-4969-875b-9a00bb9c7c6c",
            "d393kjj2-973b-3811-846c-9g33qq4c9a9f"]

}

Deployer impact
---------------

None.

Developer impact
----------------

None.

Sahara-image-elements impact
----------------------------

None.

Sahara-dashboard / Horizon impact
---------------------------------

We need to add a box to allow the user to insert the number of clusters that
will be created (default set to 1).

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  tellesmvn

Work Items
----------

* Implement the backend for creating multiple clusters
* Implement the API change
* Implement Unit tests
* Implement changes to the python-saharaclient
* Implement changes to the UI
* Update WADL file in the api-site repo

Dependencies
============

None.


Testing
=======

We will implement unit tests.

Documentation Impact
====================

The documentation needs to be updated datailing the new option.

References
==========

None.
