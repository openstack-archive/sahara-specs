..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=========================================
There and back again, a roadmap to API v2
=========================================

https://blueprints.launchpad.net/sahara/+spec/v2-api-experimental-impl

As sahara's API has evolved there have been several features introduced
in the form of routes and methods that could be crafted in a more
consistent and predictable manner. Additionally, there are several new
considerations and methodologies that can only be addressed by updating the
major version of the API. This document serves as a roadmap to implement an
experimental v2 API which will form the basis of the eventual stable version.

.. note::

    This is an umbrella specification covering many changes, there will be
    followup specifications to cover some of the more intricate details.

Problem description
===================

The current version of sahara's REST API, 1.1, contains several methodologies
and patterns that have created inconsistencies within the API and with
respect to the API Working Group's evolving guidelines[1]. Many of these
are due to the iterative nature of the design work, and some have been created
at a time before stable guidelines existed.

Examples of inconsistencies within the current API:

* Inaccurate names for method endpoints, for example "jobs" instead of
  "job-templates".

* Technology specific parameters in JSON resources, for example "oozie_job_id"
  instead of "engine_job_id".

* Improper HTTP method usage for some operations, for example using PUT for
  partial resource updates instead of PATCH.

In addition to resolving the inconsistencies in the API, a new version will
provide an opportunity to implement features which will improve the experience
for consumers of the sahara API.

Examples of features to implement in the new API:

* Micro-version support to aid in feature discovery by client applications.

* Creation of tasks endpoint and infrastructure to improve usage of
  asynchronous operations.

* HREF embedding in responses to improve resource location discovery.

These are just a few examples of issues which can be addressed in a new
major version API implementation.


Proposed change
===============

To address the creation of a new major version API, an experimental ``/v2``
endpoint should be created. This new endpoint will be clearly marked as
experimental and no contract of stability will be enforced with regards to
the content of its sub-endpoints. Changes to the ``/v2`` endpoint will be
tracked through features described in this specification, and through further
specifications which will be created to better describe the details of larger
changes.

When all the changes to the ``/v2`` endpoint have been made such that it
has a 1:1 feature compliance with the current API, and the Python sahara
client has been updated to use these new endpoints, the experimental status
of the API should be assessed with the goal of marking it as stable and
ready for public consumption.

The individual changes will be broken down into individual tasks. This will
allow the sahara team members to more easily research and implement the
changes. These efforts will be coordinated through a page on the sahara
wiki site[2].

Initial v2 commit
-----------------

The initial changes to create the ``/v2`` endpoint should also include moving
the Identity project identifier to a header named ``OpenStack-Project-ID``.
In all other respects, the endpoints currently in place for the ``/v1.1`` API
will be carried forward into the new endpoint namespace. This will create a
solid base point from which to make further changes as the new API evolves
and moves towards completion of all features described in the experimental
specifications.

Removing the project identifier from the URI will help to create more
consistent, reusable, routes for client applications and embedded HREFs.
This move will also help decouple the notion of URI scoped resources being
tied to a single project identifier.

Roadmap of changes
------------------

The following list is an overview of all the changes that should be
incorporated into the experimental API before it can be considered for
migration to stable. These changes are not in order of precedence, and can
be carried out in parallel. Some of these changes can be addressed with
simple bugs, which should be marked with ``[APIv2]`` in their names. The more
complex changes should be preceeded by specifications marked with the same
``[APIv2]`` moniker in their names. For both types of changes, the
commits should contain ``Partial-Implements: bp v2-api-experimental-impl``
to aid in tracking the API conversion process.

Overview of changes:

* Endpoint changes

  * /images/{image_id}/tag and /images/{image_id}/untag should be changed
    to follow the guidelines on tags[3].

  * /jobs should be renamed to /job-templates.

  * /job-executions should be renamed to /jobs.

  * executing a job template through the /jobs/{job_id}/execute endpoint
    should be changed to a POST operation on the new /jobs endpoint.

  * cancelling a job execution through the
    /job-executions/{job_execution_id}/cancel endpoint should be removed in
    favor of requesting a cancelled state on a PATCH to the new
    /jobs/{job_id} endpoint.

  * /job-binary-internals should be removed in favor of /job-binaries as
    the latter accepts internal database referenced items, an endpoint
    under /job-binaries can be created for uploading files(if required).

  * /job-executions/{job_execution_id}/refresh-status should be removed in
    favor of using a GET on the new /jobs/{job_id} endpoint for running
    job executions.

  * all update operations should synchronize around using PATCH instead of
    PUT for partial resource updates.

* JSON payload changes

  * hadoop_version should be changed to plugin_version.

  * oozie_job_id should be changed to engine_job_id.

  * all returned payloads should be wrapped in their type, this is currently
    true for the API and should remain so for consistency.

  * HREFs should be embedded in responses that contain references to other
    objects.

* New features

  * Identity project identifier moved to headers. This will be part of the
    initial version 2 commit but is worth noting as a major feature change.

  * Micro-version support to be added, this should be documented fully in
    a separate specification but should be based on the work done by the
    ironic[4] and nova[5] projects. Although implemented during the
    experimental phase, these microversions will not implement the backward
    compatibility features until the API has been declared stable. Once
    the API has moved into the stable phase, the microversions will only
    implement backward compatibility for version 2, and only for features
    added after the stable release.

  * Version discovery shall be improved by adding support for a "home
    document" which will be returned from the version 2 root URL. This
    document will follow the json-home[6] draft specification.
    Additionally, support for microversion discovery will be added
    using prior implementations and the API working group guidelines as
    guides.

  * Creation of an actions endpoint for clusters to provide a single
    entrypoint for operations on those clusters. This endpoint should
    initially allow operations such as scaling but will be used for
    further improvements in the future. The actions endpoint will be
    the subject a separate specification as it will describe the
    removal of several verb-oriented endpoints that currently exists,
    and the creation of a new mechanism for synchronous and asynchronous
    operations.

This list is not meant to contain all the possible future changes, but a
window of the minimum necessary changes to be made before the new API can
be declared as stable.

The move to stable for this API should not occur before the Python sahara
client has been updated to use the new functionality.

Alternatives
------------

An alternative might be to make changes to the current version API, but this
is inadvisable as it breaks the API version contract for end users.

Although the current version API can be changed, there is no way to safely
make the proposed changes without breaking backward compatibility. As the
proposed changes are quite large in nature it is not advisable to create a
"1.2" version of the API.

Data model impact
-----------------

Most of these changes will not require modifications to the data model. The
two main exceptions are the payload name changes for ``hadoop_version`` and
``oozie_job_id``. As the data model will continue to be used for the v1.1
API until it is deprecated, it is not advisable to rename these fields at
this time. When the v2 API has been made stable, and the v1.1 API has been
deprecated, these fields should be revisisted and changed in the data model.

During the experimental phase of the API, these translations will occur in
the code that handles requests and responses. After the API has transitioned
to production mode, migrations should be created to align the data models
with the API representations and translations should be created for the
older versions only as necessary. As the older version API will eventually
be deprecated, these changes should be scheduled to coincide with that
transition.

REST API impact
---------------

As this specification is addressing a high level change of the API, the
following changes are enumerated in brief. Full details should be created
for changes that will require more than just renaming an endpoint.

* creation of /v2 root endpoint

* removal of {tenant_id} from URI, to be replaced by
  ``OpenStack-Project-ID`` header on all requests.

* removal of POST to /images/{image_id}/tag

* removal of POST to /images/{image_id}/untag

* creation of GET/PUT/DELETE to /images/{image_id}/tags, this should be
  followed with a specification describing the new tagging methodology.

* creation of GET/PUT/DELETE to /images/{image_id}/tags/{tag_id}, this
  should also be in the previously mentioned specification on tagging.

* move operations on /jobs to /job-templates

* move operations /job-executions to /jobs

* removal of POST to /jobs/{job_id}/execute

* creation of POST to /jobs, this should be defined in a specification
  about restructuring the job execution endpoint.

* creation of jobs via the /jobs endpoint should be transitioned away
  from single input and output fields to use the newer job configuration
  interface[7].

* removal of GET to /job-executions/{job_execution_id}/cancel

* creation of PATCH to /jobs/{job_id}, this should be defined in the
  specification about restructuring the job execution endpoint.

* removal of GET to /job-executions/{job_execution_id}/refresh-status

* removal of all /job-binary-internals endpoints with their functionality
  being provides by /job-binaries, this may require creating a separate
  sub-endpoint for uploading.

* refactor of PUT to /node-group-templates/{node_group_template_id} into
  PATCH on same endpoint.

* refactor of PUT to /cluster-templates/{cluster_template_id} into PATCH on
  same endpoint.

* refactor of PUT to /job-binaries/{job_binary_id} into PATCH on same
  endpoint.

* refactor of PUT to /data-sources/{data_source_id} into PATCH on same
  endpoint.

Other end user impact
---------------------

In the experimental phase, this change should have no noticeable affect on
the end user. Once the API has been declared stabled, users will need to
switch python-saharaclient versions as well as upgrade their horizon
installations to make full use of renamed features.

Deployer impact
---------------

During the experimental phase, this change will have no effect on deployers.

When the API reaches the stable phase, deployers will be responsible for
upgrading their installations to ensure that sahara and python-saharaclient
are upgraded as well as changing the service catalog to represent the
base endpoint.

Developer impact
----------------

As this change is targeted for experimental work, developers should know
that the details of the v2 API will be constantly changing. There is no
guarantee of stability.

Sahara-image-elements impact
----------------------------

None

Sahara-dashboard / Horizon impact
---------------------------------

This change should not require changes to horizon as many of the primitives
that are changing already display the proper names, for example
"Job Templates". When this change moves to the stable phase, horizon should
be re-evaluated.


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  mimccune (Michael McCune)

Other contributors:


Work Items
----------

The main work item for this specification is the initial v2 commit.

* create v2 endpoint
* create code to handle project id in headers
* create mappings to current endpoints


Dependencies
============

This change should not require new dependencies.


Testing
=======

Unit tests will be created to exercise the new endpoints. Additionally, the
gabbi[8] testing framework should be investigated as a functional testing
platform for the REST API.

To improve security testing, tools such as Syntribos[9] and RestFuzz[10]
should be investigated for use in directed testing efforts and as possible
gate tests.

These investigations should result in further specifications if sufficient
results are discovered to warrent their creation as they will deal with
new testing modes for the sahara API server.

As the v2 API reaches stable status, and the python-saharaclient has been
ported to use the new API, the current functional tests should provide the
necessary framework to ensure successful end-to-end testing.


Documentation Impact
====================

During the experimental phase, this work will not produce documentation. As
the evaluation for stable approaches there will need to be a new version of
the WADL files for the api-ref[11] site, if necessary. There is the
possibility that this site will change its format, in which case these new
API documents will need to be generated.

Further, the v2 API should follow keystone's model[12] of publishing
the API reference documents in restructured text format to the specs
repository. This would make the API much easier to document and update as
new specification changes could also propose their API changes to the same
repo. Also, the WADL format is very verbose and the future of this format is
under question within the OpenStack documentation community[13]. The effort
to make accurate documentation for sahara's API should also include the
possibility of creating Swagger[14] output as the v2 API approaches stable
status, this should be addressed in a more separate specification as that
time approaches.


References
==========

[1]: http://specs.openstack.org/openstack/api-wg/#guidelines

[2]: https://wiki.openstack.org/wiki/Sahara/api-v2

[3]: http://specs.openstack.org/openstack/api-wg/guidelines/tags.html

[4]: http://specs.openstack.org/openstack/ironic-specs/specs/kilo-implemented/api-microversions.html

[5]: http://specs.openstack.org/openstack/nova-specs/specs/kilo/implemented/api-microversions.html

[6]: https://tools.ietf.org/html/draft-nottingham-json-home-03

[7]: http://specs.openstack.org/openstack/sahara-specs/specs/liberty/unified-job-interface-map.html

[8]: https://github.com/cdent/gabbi

[9]: https://github.com/openstack/syntribos

[10]: https://github.com/redhat-cip/restfuzz

[11]: http://developer.openstack.org/api-ref.html

[12]: https://github.com/openstack/keystone-specs/tree/master/api

[13]: http://specs.openstack.org/openstack/docs-specs/specs/liberty/api-site.html

[14]: https://github.com/swagger-api/swagger-spec

Liberty summit etherpad,
https://etherpad.openstack.org/p/sahara-liberty-api-v2

Mitaka summit etherpad,
https://etherpad.openstack.org/p/sahara-mitaka-apiv2
