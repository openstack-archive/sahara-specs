..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Support multi-worker Sahara API deployment
==========================================

https://blueprints.launchpad.net/sahara/+spec/sahara-api-workers

Add support of multi-worker deployment of Sahara API.

Problem description
===================

Currently Sahara API uses one thread with wsgi application. This means that
API can service only one request at a time. Some requests (e.g. cluster scale)
require synchronous requests to Sahara engine (using message queue). This
means that Sahara API will not be able to service other requests until full
round trip finished.

Also, multi-threaded solution gives much more options for performance tuning.
It could allow to utilize more server CPU power.

Most of OpenStack services support several workers for API.

Proposed change
===============

The ideal solution for that would be migration to Pecan and WSME (
https://etherpad.openstack.org/havana-common-wsgi) with multi-threading
support. Although this will require a lot of work and there is no much
pressure to do that.

This spec suggests simple solution of specific problem without much
refactoring of existing code.

So, the solution is:
1. Leave current wsgi implementation
2. Leave current socket handling
3. Run wsgi server in several threads/processes
4. Implement only children processes management, leave all existing code as is.

Children processes management will include:

1. Handling of children processes, restart of dead processes
2. Proper signals handling (see https://bugs.launchpad.net/sahara/+bug/1276694)
3. Graceful shutdown
4. Support of debug mode (with green threads instead of real threads)

Things that will NOT be included:
1. Config reload / API restart

Alternatives
------------

Migrate to Pecan and WSME first.

Implementation details
----------------------

Most OpenStack services use deprecated oslo wsgi module. It has tight
connections with oslo services module.

So, there are three options here:

1. Use deprecated oslo wsgi module. (Bad, since module is deprecated)
2. Use oslo services module, but write all wsgi stuff ourselves (or copy-paste
   from other project).
3. Write minimum code to make server start multi-worker (e.g. see how it is
   implemented in Heat).

I propose going with the option 3. There is no much sense spending resources
for code, that will be replaced anyway (we will definitely migrate to Pecan
some day).

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

One more configuration parameter.

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

* Implement feature
* Document feature

Dependencies
============

None

Testing
=======

Manually. Probably CI could be changed to run different tests in
different modes.

Documentation Impact
====================

Need to be documented.

References
==========

None.