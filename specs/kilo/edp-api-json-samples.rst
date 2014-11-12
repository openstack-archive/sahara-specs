..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=================================
JSON sample files for the EDP API
=================================

https://blueprints.launchpad.net/sahara/+spec/edp-api-json-samples

Provide sample JSON files for all EDP Sahara APIs to facilitate ease of
use by command-line users.


Problem description
===================

As an End User who prefers the Sahara CLI to its UI, I want a set of
pre-constructed example JSON payloads for the Sahara EDP API so that I can
easily learn the expected API signatures and modify them for my use.


Proposed change
===============

Example JSON payloads will be added to the directory
``sahara/etc/edp-examples/json-api-examples/v1.1``, with a subpath for each
relevant manager (data_source, job, job_binary, job_binary_internals, and
job_execution.) It is intended that examples for future API versions will
follow this path structure.

Each file will be named after the pattern: ``method_name.[variety.]json``,
where variety is optional and will be used to describe independently useful
variations in payloads for one method (as in varying engines underlying
data sources or processing jobs.)

Alternatives
------------

A tool could conceivably be created to generate template payloads from the
jsonschemata themselves. However, as the core use of this change is to
provide immediately available, semantically valid payloads for ease of
adoption, it is proposed that providing raw examples will better meet the
perceived user need.

It would also be possible to package these examples directly with the
python-saharaclient repository, an option which has much to recommend it.
However, as these examples are globally useful to any non-UI interface,
as they are reliant on the jsonschemata in the core repository for testing,
and as the extant etc/edp-examples path is a natural home for them,
placing them in the sahara repository itself seems indicated.

Data model impact
-----------------

None.

REST API impact
---------------

None.

Other end user impact
---------------------

None, though it is intended that the payloads may be used via the
python-saharaclient.

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
  egafford

Other contributors:
  tmckay (primary review)

Work Items
----------

* Payload generation: data_source
* Payload generation: job
* Payload generation: job_binary
* Payload generation: job_binary_internals
* Payload generation: job_execution
* Addition of schema validation unit tests for all the above.


Dependencies
============

None.


Testing
=======

After discussion with croberts and tmckay, it is proposed that integration
testing is in this case unnecessary; these examples constitute documentation.
While exhaustive testing is possible in this case, the resultant bloat of
the CI build would be disproportionate to the utility of the change.

Unit testing will validate that these resources pass schema validation for
their intended APIs.


Documentation Impact
====================

This task is itself a documentation effort. A README.rst will be provided
in place, in keeping with pre-existent etc/edp-examples.


References
==========

* https://etherpad.openstack.org/p/kilo-summit-sahara-edp
