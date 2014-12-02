..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Spec - Add support for filtering results
==========================================

Blueprints:

Sahara service:  https://blueprints.launchpad.net/sahara/+spec/enable-result-filtering

Client library:  https://blueprints.launchpad.net/python-saharaclient/+spec/enable-result-filtering

Horizon UI:  https://blueprints.launchpad.net/horizon/+spec/data-processing-table-filtering

As result sets can be very large when dealing with nontrivial openstack
environments, it is desirable to be able to filter those result sets by
some set of field constraints.  This spec lays out how Sahara should support
such queries.  Changes will be required in the API, client library,
and in the Horizon UI.

Problem description
===================

The original use case for this change came up when we wanted to provide a
way to filter some of the tables in the UI.  In order to make that filtering
possible, either the UI has to perform the filtering (inefficient since
large result sets would still be returned by the api) or the api/client
library need to provide support for such filtering.  Upon review of other
api/client libraries, it looks like most, if not all,
provide filtering capabilities.  Sahara should also make filtering possible.

Proposed change
===============

In order to make filtering possible, each of the "list()" methods in the
client library should be extended to take an optional (default=None)
parameter, "search_opts".  "search_opts" should be a dict that will include
one or more {field:query} entries.  Pagination can also be handled through
search_opts by allowing for the setting of page/page_size/max results.

In addition, the REST api needs to be extended for the list() operations to
support query syntax.  This will require changes to how Sahara currently
does its database queries to include query options.  Currently,
they are set to just return all entries for each call.

Alternatives
------------

Filtering could be done entirely in the UI, but that is a wasteful way to do
it since each library/api call will still return the full result set.
Also, any users of the REST api or the client library would not benefit
from this functionality.

It would also be possible to just implement filtering at the client library
level, but for the same reasons that doing it in the UI is undesirable,
this approach is also suboptimal.

Data model impact
-----------------

No changes to the model itself should be required.  Only changes to how we
query the database are needed.

REST API impact
---------------

The rest api methods for the list operations will need to be changed to
support query-style parameters.  The underlaying database access methods
will need to be changed from the basic "get all" functionality to support a
`**kwargs` parameter (this is already done in cluster_get_all(),
but needs to be done for the other methods).

There are a pair of special cases to be considered for filtering on the Job
Executions table.  The "job" and "cluster" columns actually contain
information that are not part of the job execution object.  For filters on
those fields, a field value of "job.name" or "cluster.name" should be passed
down.  That will trigger the database query to be joined to a filter on the
name property of either the job or cluster table.


Other end user impact
---------------------

Users of the REST API, client library and Horizon UI will be able to filter
their result sets via simple queries.

Deployer impact
---------------

These changes should not affect deployment of the Sahara service.

Developer impact
----------------

No developer impact.

Sahara-image-elements impact
----------------------------

No sahara-image-elements imbact.

Sahara-dashboard / Horizon impact
---------------------------------

Once the REST API and client library changes are in place,
the Horizon UI will be modified to allow for filtering on the following
tables: Clusters, Cluster Templates, Node Group Templates, Job Executions,
Jobs.  It would also be possible to filter any of our tables in the future
without any other API/client library changes.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  croberts (Horizon UI)

Other contributors:
  tmckay (REST API/sahara.db.api)
  croberts (Client Library)


Work Items
----------
These may be able to be worked on in parallel, but they must be released in
the order they are given below.

1. REST API implementation
2. Client library implementation
3. Horizon UI implementation


Dependencies
============

None


Testing
=======

There should be tests added against the REST interface,
client library and Horizon UI to test the filtering capabilities being added.


Documentation Impact
====================

Docs for the REST API will need to be updated to reflect the querying
functionality.

Docs for the client library will need to be updated to reflect the querying
functionality.

The dashboard user guide should be updated to note the table filtering
functionality that will be added.


References
==========

Sahara service:  https://blueprints.launchpad.net/sahara/+spec/enable-result-filtering

Client library:  https://blueprints.launchpad.net/python-saharaclient/+spec/enable-result-filtering

Horizon UI:  https://blueprints.launchpad.net/horizon/+spec/data-processing-table-filtering
