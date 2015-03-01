..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===========================================
Use trusts for cluster creation and scaling
===========================================

https://blueprints.launchpad.net/sahara/+spec/cluster-creation-with-trust

Creation of cluster can be a pretty long operation. Currently we use sessions
that usually expire in less than one hour. So, currently it is impossible to
create a cluster that requires more than one hour for spawn.

Sahara could get trust from user and use it whenever it is needed for cluster
creation or scaling.

Problem description
===================

Sahara communicates with OpenStack services using session provided by user.
Session is created by keystone for comparably small amount of time. Creation
of large cluster could require more than session life time. That's why Sahara
will not be able to operate cluster at the end of the process.

Proposed change
===============

Sahara could perform all operations with cluster using trust obtained from
user. Now trusts are used for termination of transient cluster. The same
logic could be used for all operations during cluster creation and scaling.

Since we still support keystone v2, the option should be configurable. I
suggest making it enabled by default.

The proposed workflow:

1. User requests cluster creation or scaling
2. Sahara creates trust to be used for OpenStack operation. Trust is stored in
   memory only (probably context is the good place to put it). No
   serialization to DB.
3. Sahara finishes cluster provisioning and deletes trust.

For safety reasons created trusts should be limited by time, but their life
time should be sufficient for cluster creation. Parameter in config file with
1 day default should work well.

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

None.

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

Other contributors:
  None

Work Items
----------

* Add option to Sahara config
* Implement new authentication strategy
* Document new behavior

Dependencies
============

None.

Testing
=======

Manually. CI will cover the feature.

Documentation Impact
====================

Need to be documented.

References
==========

None.