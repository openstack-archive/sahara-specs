..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=====================
Force Delete Clusters
=====================

https://blueprints.launchpad.net/sahara/+spec/sahara-force-delete

Let's resolve a long-standing complaint.

Problem description
===================

An oft-reported issue is that a Sahara cluster hangs on deletion. This can be
a major user headache, and also looks really bad. Usually the problem is not
the fault of Sahara, and rather the fault of some undeletable resource.

Proposed change
===============

We will make use of Heat's stack abandoning feature. Essentially, this means
the stack will get deleted but leave resources in place. It's a last-ditch
effort to be made when undeleteable (or slow-to-delete) resources are present
in the stack.

This will be done through a new addition to Sahara API which wraps Heat stack
abandon. And it's probably best to release as part of APIv2 rather than
continue to break the rules that we have broken so often for APIv1.1.

Note that even though Heat stack abandon does not clean up any resources, we
should not have Sahara try to clean them up manually.

The above point is justified by two things:

* This gerrit comment [0]
* If abandoning is needed, it's really hard to delete the resource anyway

It's best to create an API which wraps stack abandon, rather than just telling
users to use abandon directly, because there's other cleanup to do during
cluster delete. See [1] for more info.

The change will enable the following workflow: force delete gets called and
cleans up cluster as best it can, then user gets to handle orphaned resources
themselves. Thanks to explicit abandon through Sahara API, users are always
encouraged to make sure stack is in deleting state first, so that amount of
orphaned resources is minimized.

With regards to the above point, it is absolutely crucial that we do not
simply enhance the regular delete call to include an abandon call. There are
two key reasons to avoid that:

* In normal operation, Heat stack delete does retry
* In normal use, probably the user wants cluster to stay in deleting state
  until resources actually gone: force delete is just for emergencies


Alternatives
------------

Just tell users/operators to use Heat's stack abandon manually. That's not a
great choice for the reasons discussed above.

Data model impact
-----------------

None

REST API impact
---------------

We'll add the following endpoint:

.. sourcecode:: console

        DELETE /v2/clusters/{cluster_id}/force

..

* It'll give 204 on success just like regular DELETE.
* It'll give the usual 4xx errors in the usual ways.
* It'll give 503 (on purpose)  when Heat stack abandon is unavailable.
* Request body, headers, query, etc are the same as regular DELETE.

Again, best practice says make this a v2 exclusive, rather than further dirty
the existing v1.1 API.

Other end user impact
---------------------

Need to extend Saharaclient API bindings and OSC in order to support the new
API methods.

Deployer impact
---------------

They need to enable_stack_abandon=True in heat.conf and make sure that their
cloud policy does allow users to do such an operation.

We could try to do something fancy with RBAC and trusts so that Sahara service
user may abandon Heat stacks on behalf of the user, when the operator wishes
to restrict stack abandon. But it might not be worth messing with that...

Developer impact
----------------

None

Image Packing impact
--------------------

None

Sahara-dashboard / Horizon impact
---------------------------------

Yes, expose the functionality in the UI. We could put some warnings about
force-delete's implications as well.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  Jeremy Freudberg

Other contributors:
  None

Work Items
----------

* Remove the last bit of direct engine code
  (without this change, even an abandoned stack may still be stuck, as Sahara
  engine is trying manual cleanup; it also entails that after this change is
  made, the bug of fully deleted stack but cluster still in deleting state is
  essentially resolved...)
* Create the new API bits and corresponding operation
* Add functionality to client and dashboard

Dependencies
============

None

Testing
=======

Probably scenario tests are not strictly needed for this feature.

Beyond the obvious unit tests, there will also be updates to the API tests in
the tempest plugin.

Documentation Impact
====================

Nothing out of the ordinary, but important to keep in mind both operator and
developer perspective.

References
==========

[0] https://review.openstack.org/#/c/466778/20/sahara/service/engine.py@276

[1] https://github.com/openstack/sahara/blob/master/sahara/service/ops.py#L355
