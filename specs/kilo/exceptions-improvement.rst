..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Exceptions improvement
==========================================

https://blueprints.launchpad.net/sahara/+spec/exceptions-improvement

This specification proposes to add identifiers for every raised Sahara
exception, so that they can be easily found in logs.

Problem description
===================

Now it's hard to find an error in logs, especially if there are a lot of
errors of the same type which occurs when a lot of operations are executed
simultaneously. They can produce a bunch of similar exceptions (error code
doesn't help in this kind of situation).

It would be nice to have an opportunity to find exceptions by unique
identifiers. This identifiers will be found in Horizon tab with events that
will be implemented in this spec: https://review.openstack.org/#/c/119052/.

Proposed change
===============

Support Features:

* Every error that has been raised during the workflow will have, besides of
  error message, uuid property, whereby error can be found in logs easily.

For example, NotFoundException will leave in logs:

.. sourcecode:: console

NotFoundException: Error ID: 7a229eda-f630-4153-be03-d71d6467f2f4
Object 'object' is not found

..

Alternatives
------------

None

Data model impact
-----------------

None

REST API impact
---------------

None

Other end user impact
---------------------

None

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

None

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  apavlov-n

Other contributors:
  sreshetniak

Work Items
----------

* Adding ability to generate unique identifiers for SaharaException class
* Change messages of Sahara exceptions so that all of them contain
  identifier.

Dependencies
============

None

Testing
=======

None

Documentation Impact
====================

None

References
==========

None
