..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=================================
Decommission of specific instance
=================================

https://blueprints.launchpad.net/sahara/+spec/decommission-specific-instance

When facing issues with a cluster it can be useful to remove an specific
instance. The way that Sahara is constructed today allows the user to scale a
cluster down but it will choose a random instance from the selected node group
to be removed. We want to give the users the opportunity to choose which
instance(s) he/she would like to remove from the cluster.

Problem description
===================

Users may need to remove specific node to make cluster healthier. This is not
possible today.

Proposed change
===============

We will add the possibility for the user to choose the specific instance to
remove from the cluster.

After selecting the node group from which the instance will be removed the user
will be allowed to choose the instance or instances to be removed.

We will also allow wildcard removal, if the user wants to randomly select the
instance he can just leave it blank as well as if more than one instance is
being deleted the user will be able to choose each instance to be deleted or
just a subset and Sahara will choose the rest.

Alternatives
------------

Keep randomly selecting instance to be scaled down.

Data model impact
-----------------

None

REST API impact
---------------

We will change the body request for scale cluster.

Currently the body should be like this:

{
    "add_node_groups": [
        {
            "count": 1,
            "name": "b-worker",
            "node_group_template_id": "bc270ffe-a086-4eeb-9baa-2f5a73504622"

        }

    ],

    "resize_node_groups": [
        {
            "count": 4,
            "name": "worker"

        }

    ]

}

We will change the second part to support an extra parameter:

{
    "add_node_groups": [
        {
            "count": 1,
            "name": "b-worker",
            "node_group_template_id": "bc270ffe-a086-4eeb-9baa-2f5a73504622"

        }

    ],

    "resize_node_groups": [
        {
            "count": 4,
            "name": "worker",
            "instances": ["instance_id1", "instance_id2"]

        }

    ]

}

In case the user does not specify instances to be removed the parameter will
not be passed and we will act on removing with the current approach.

Other end user impact
---------------------

CLI command will have a new option to select instances.

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

We will need to add a place for user to select the instances to be removed. It
can be after the NG selection and we add the number of selector of instances to
be removed with the option to leave it blank.


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  Telles Nobrega

Other contributors:
  None

Work Items
----------

* Add the possibility to select an instance when scaling down
* Add CLI option to select instances to be removed
* Add UI option to select instances to be removed
* Unit tests
* Documentation

Dependencies
============

None

Testing
=======

Unit tests will be needed.

Documentation Impact
====================

Nothing out of the ordinary, but important to keep in mind both user and
developer perspective.


References
==========

None
