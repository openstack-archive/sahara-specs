..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

====================================
Retry of all OpenStack clients calls
====================================

https://blueprints.launchpad.net/sahara/+spec/clients-calls-retry

This specification proposes to add ability of retrying OpenStack clients calls
in case of occasional errors occurrence.

Problem description
===================

Sahara uses a bunch of OpenStack clients to communicate with other OpenStack
services. Sometimes during this clients calls can be occurred occasional
errors that lead to Sahara errors as well. If you make a lot of calls, it may
not be surprising if one of them doesn't respond as it should - especially for
a service under heavy load.

You make a valid call and it returns a 4xx or 5xx error. You make the same
call again a moment later, and it succeeds. To prevent such kind of failures,
all clients calls should be retried. But retries should be done only for
certain error codes, because not all of the errors can be avoided just with
call repetition.

Proposed change
===============

Swift client provides the ability of calls retry by its own. So, only number of
retries and retry_on_ratelimit flag should be set during client initialisation.

Neutron client provides retry ability too, but repeats call only if
``ConnectionError`` occurred.

Nova, Cinder, Heat, Keystone clients don't offer such functionality at all.

To retry calls ``execute_with_retries(method, *args, **kwargs)`` method will be
implemented. If after execution of given method (that will be passed with first
param), error occurred, its ``http_status`` will be compared with http statuses
in the list of the errors, that can be retried. According to that, client call
will get another chance or not.

There is a list of errors that can be retried:

* ``REQUEST_TIMEOUT (408)``
* ``OVERLIMIT (413)``
* ``RATELIMIT (429)``
* ``INTERNAL_SERVER_ERROR (500)``
* ``BAD_GATEWAY (502)``
* ``SERVICE_UNAVAILABLE (503)``
* ``GATEWAY_TIMEOUT (504)``

Number of times to retry the request to clients before failing will be taken
from ``retries_number`` config value (5 by default).

Time between retries will be configurable (``retry_after`` option in
config) and equal to 10 seconds by default. Additionally, Nova client provides
``retry_after`` field in ``OverLimit`` and ``RateLimit`` error classes, that
can be used instead of config value in this case.

These two config options will be under ``timeouts`` config group.

All clients calls will be replaced with ``execute_with_retries`` wrapper.
For example, instead of the following method call

.. sourcecode:: python

    nova.client().images.get_registered_image(id)

it will be

.. sourcecode:: python

    execute_with_retries(nova.client().images.get_registered_image, id)

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

Work Items
----------

* Adding new options to Sahara config;
* ``execute_with_retries`` method implementation;
* Replacing OpenStack clients call with ``execute_with_retries`` method.

Dependencies
============

None

Testing
=======

Unit tests will be added. They will check that only specified errors will
be retried

Documentation Impact
====================

None

References
==========

None
