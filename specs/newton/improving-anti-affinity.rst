..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=================================================================
Improve anti-affinity behavior for cluster creation
=================================================================

https://blueprints.launchpad.net/sahara/+spec/improving-anti-affinity

Enable sahara to distribute node creation in a more equitable manner with
respect to compute hardware affinity.


Problem description
===================

Current anti-affinity in sahara allows nodes in the anti-affinity group,
equal to the number of hypervisors
(https://bugs.launchpad.net/sahara/+bug/1426398).

If the number of nodes in the anti-affinity group are more than the number of
hypervisors, sahara throws an error.


Proposed change
===============

User will be able to define a ratio i.e number of nodes per hypervisor
when requesting anti-affinity for a process.

The ratio would be a field while creating a cluster if user selects
anti-affinity

Based on the ratio given by the user and number of nodes, more server
groups will be created.

Number of server groups would be equal to the number of nodes per hypervisor.

In terms of heat templates, the server groups would be created while
serializing the resources if anti-affinity is enabled for that cluster.

Instances would be allocated to those server groups while serializing the
instance using "group" property of "scheduler_hints" which will be set to
different server group for each instance in round robin fashion.

For allocation of server groups, following changes would be required:

* Create a parameter named SERVER_GROUP_NAMES of type list in
  the OS::Heat::ResourceGroup resource

* Store the server group name for each instance in the node group in
  this parameter. So the size of the parameter list would be equal to
  the number of instances in the node group

* Now the instance with index i would belong to the server group name
  stored at SERVER_GROUP_NAMES[i]

* This parameter will then be accessed from the scheduler hints

So in the node group template, scheduler hints will look like this,

.. code-block:: python

   "scheduler_hints": {
    "group": {
     "get_param": [SERVER_GROUP_NAMES, {"get_param": "instance_index"}]
    }
   }


E.g

A = Number of hypervisors = 5

B = Total number of nodes in the a-a group = 10

C = Number of nodes per hypervisor = nodes:hypervisor = 2

Number of server groups = C = 2

Nodes would be distributed in each of the created server groups in round-robin
fashion.

Although, placement of any node in any of the server groups does not matter
because all the nodes are anti-affine.

In case the ratio given by the user in the above example is 1, user will
still get an error which will be thrown by nova.

We won't allow old clusters to scale with new ratio

When a user requests to scale a cluster after the ratio has changed or requests
a new ratio on an existing cluster, an error would be thrown saying "This
cluster was created with X ratio, but now the ratio is Y. You will need to
recreate".

Alternatives
------------

None

Data model impact
-----------------

Ratio would be a field in the cluster object

REST API impact
---------------

None

Other end user impact
---------------------

The change would provision the instances without any error even in case of more
nodes in the anti-affinity group than the number of hypervisors if user defines
the ratio correctly.

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

Yes a field has to be added in the sahara dashboard for collecting the ratio.
The field will be displayed only when anti-affinity is selected.


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  akanksha-aha


Work Items
----------

* Add a ratio field in sahara-dashboard

* Add the same field in sahara wherever required (Data Access Layer)

* Add a new API which creates more server groups when required

* Write Unit tests and run those tests

* Write documentation


Dependencies
============

None


Testing
=======

Will need to write unit tests


Documentation Impact
====================

Need to add about improved anti-affinity behavior


References
==========

None
