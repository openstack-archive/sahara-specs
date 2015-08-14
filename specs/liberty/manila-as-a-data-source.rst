..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===============================
Manila as a runtime data source
===============================

https://blueprints.launchpad.net/sahara/+spec/manila-as-a-data-source

Using manila nfs shares and the mounting features developed for use with
job binaries, it should be feasible to use nfs shares to host input
and output data. This will allow data to be referenced through local
filesystem paths as a another simple alternative to hdfs or swift storage.


Problem description
===================

The work has already been done to support mounting of manila nfs shares at
cluster provisioning time or automounting shares for EDP job binaries with a
url of the form *manila://<share-id>/path*. Additionally, the Hadoop filesystem
APIs already support *file:///path* urls for referencing the local filesystem.

Sahara can build on these existing features by allowing
*manila://<share-id>/path* urls for data sources, automounting shares
referenced by data sources when necessary, and generating the correct
local filesystem urls for EDP jobs at runtime.

Some of the benefits of this approach are:

* parity for job binaries and data sources in regards to manila shares
* the ability to store job binaries and data sources in the same share
* the flexibility to add a new data share to a cluster at any time
* the ability to operate on data from a cluster node using native OS tools
* works on any node where *mount -t nfs ...* is supported
* lays the groundwork for other manila share types in the future

The problem can be divided into three high-level items:

* Add a *manila* data source type with validation of *manila://* urls
* Translate *manila://* urls to *file://* urls for use at job runtime
* Call the existing automounting methods when a data source references
  a new share

Note, automounting and url translation will only work for manila shares
referenced by data source objects. A *manila://* url embedded as a literal
in a job config, param, or arg will be ignored.  It will not be translated
to a *file://* url by Sahara and it will not cause automounting. However,
there is a precedent for this -- Sahara currently has other features that
are only supported on data source objects, not on literal urls. (It may
be possible to remove these limitations in the future through greater
use of the unified job mapping interface recently introduced).

Proposed change
===============

A *manila* data source type will be added to the JSON schema for data sources,
with appropriate validation of *manilla://* urls.

The existing code in *sahara/service/edp/binary_retrievers/manila_share.py*
that supports path name generation and automounting of manila nfs shares for
job binaries will be refactored and broken up between
*sahara/service/edp/job_utils.py* and *sahara/service/shares.py*. The essential
implementation is complete, but this logic needs to be callable from multiple
places and in different combinations to support data sources.

Currently, all data source urls are returned to the EDP engines from
*get_data_sources()* and *resolve_data_source_references()* in *job_utils.py*.
The returned urls are recorded in the job_execution object and used by the
EDP engine to generate the job on the cluster. These two routines will be
extended to handle manila data sources in the following ways:

* Mount a referenced nfs share on the cluster when necessary. Since an
  EDP job runs on multiple nodes, the share must be mounted to the whole
  cluster instead of to an individual instance

* Translate the *manila://* url to a *file://* url and return both urls
  Since the submission time url and the runtime url for these data
  sources will be different, both must be returned. Sahara will record
  the submission time url in the job_execution but use the runtime url
  for job generation

Alternatives
------------

Do not support *manila://* urls for data sources but support data hosted on nfs
as described in

https://review.openstack.org/#/c/210839/

However, these features are complementary, not mutually exclusive, and most of
the appartus necessary to make this proposal work already exists.

Data model impact
-----------------

None

REST API impact
---------------

None (only "manila" as a valid data source type in the JSON schema)

Other end user impact
---------------------

None

Deployer impact
---------------

Obviously, if this feature is desired then the manila service should be running

Developer impact
----------------

None

Sahara-image-elements impact
----------------------------

None (nfs-utils element is already underway)

Sahara-dashboard / Horizon impact
---------------------------------

Sahara needs a manila data source type on the data source creation form


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  tmckay

Other contributors:
  croberts, egafford

Work Items
----------

* Add manila data source type to the JSON schema
* Allow submission time and runtime urls to differ
  (https://review.openstack.org/#/c/209634/3)
* Refactor binary_retrievers/manila_share.py
* Extend job_utils get_data_sources() and resolve_data_source_references()
  to handle manila:// urls
* Add manila data source creation to Horizon
* Modify/extend unit tests
* Documentation

Dependencies
============

https://blueprints.launchpad.net/sahara/+spec/manila-as-binary-store


Testing
=======

Unit tests.

Eventually, as with job binaries, this can be tested with integration
tests if/when we have manila support in the gate


Documentation Impact
====================

Discussion of the manila data source type should be added to any
sections we currently have that talk about data being hosted in swift of hdfs.

Additionally, we should consider adding information to the Sahara section
of the security guide on the implications of using manila data shares.

If the security guide or the manila documentation contains a section on
security, this probably can be a short discussion from a Sahara perspective
with a link to the security info. If there isn't such a section currently, then
probably there should be a separate CR against the security guide to create a
section for Manila.

References
==========
