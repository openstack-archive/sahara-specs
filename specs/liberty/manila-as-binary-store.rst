..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

====================================
Addition of Manila as a Binary Store
====================================

https://blueprints.launchpad.net/sahara/+spec/manila-as-binary-store

Network and distributed filesystems are a useful means of sharing files among
a distributed architecture. This core use case makes them an excellent
candidate for storage of job binaries.

Problem description
===================

While internal database storage and Swift storage are sensible options for
binary storage, the addition of Manila integration allows it as a third option
for job binary storage and retrieval. This specification details how this
will be implemented.

Proposed change
===============

The scheme ``manila`` will be added as an option in creation of job binaries.
URLs will take the form:

``manila://{share_id}/absolute_path_to_file``

URLs stored as binary locations can be passed through the following
validations:

1) The manila share id exists.
2) The share is of a type Sahara recognizes as a valid binary store.

Because manila shares are not mounted to the control plane and should not be,
we will not be able to assess the existence or nonexistence of files intended
to be job binaries.

We can, however, assess the share type through Manila. For the initial
reference implementation, only NFS shares will be permitted for this
use; other share types may be verified and added in later changes.

For this binary type, Sahara will not retrieve binary files and copy them into
the relevant nodes; they are expected to be reachable through the nodes' own
filesystems. Instead, Sahara will:

1) Ensure that the share is mounted to the appropriate cluster nodes (in the
   Oozie case, the node group containing Oozie server; in the Spark case, the
   node group containing Spark Master, etc.) If the share is not already
   mounted, Sahara will mount the share to the appropriate node groups using
   the mechanism described by blueprint mount-share-api (at the default path)
   and update the cluster's DB definition to note the filesystem mount.
2) Replace ``manila://{share_id}`` with the local filesystem mount point, and
   use these local filesystem paths to build the workflow document or job
   execution command, as indicated by engine. Job execution can then take
   place normally.

It is notable that this specification does not cover support of Manila
security providers. Such support can be added in future changes, and should
not affect this mechanism.

Alternatives
------------

While verification of paths on binary creation would be ideal, mounting tenant
filesystems (in the abstract, without a cluster necessarily available) is a
prohibitive security concern that outweighs this convenience feature (even
if we assume that networking is not an issue.)

We could also create a more general ``file://`` job binary scheme, either in
addition to ``manila://`` or as a replacement for it. However, this would not
particularly facilitate reuse among clusters (without a number of manual steps
on the user's part) or allow auto-mounting when necessary.

We could also opt to simply raise an exception if the share has not already
been mounted to the cluster by the user. However, as the path to automatic
mounting is clear and will be reasonably simple once the mount-share-api
feature is complete, automatic mounting seems sensible for the initial
implementation.

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

Manila will be added as a visible job binary storage option; no other changes.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  egafford

Secondary assignee/reviewer:
  croberts

Work Items
----------

* Validation changes (URL scheme).
* Integration with mounting feature.
* Creation of "job binary retriever" strategy for Manila (which will mostly
  no-op, given the strategy above).
* Modification of workflow and execution command code to facilitate this flow.
* Horizon changes (in separate spec).
* Documentation.

Dependencies
============

None.

Testing
=======

Unit testing is assumed; beyond this, full integration testing will depend on
the feasibility of adding a manila endpoint to our CI environment. If this is
feasible, then our testing path becomes clear; if it is not, then gated
integration testing will not be possible.

Documentation Impact
====================

This feature will require documentation in edp.rst.

References
==========

See https://wiki.openstack.org/wiki/Manila/API if unfamiliar with manila
operations.
