..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==================================================
Allow 'is_public' to be set on protected resources
==================================================

Since *is_public* is meta-information on all objects rather
than content, sahara should allow it to be changed even if *is_protected*
is True. This will make it simpler for users to share protected
objects.

https://blueprints.launchpad.net/sahara/+spec/allow-public-on-protected


Problem description
===================

Currently checks on *is_protected* prevent any field in an object
from being modified. This guarantees that the content of the object
will not be changed accidentally.

However, *is_public* is an access control flag and does not really
pertain to object content. In order to share a protected object,
a user must currently set *is_protected* to False while making
the change to *is_public*, and then perform another operation
to set the *is_protected* flag back to True.

Proposed change
===============

As a convenience, allow *is_public* to be modified even if *is_protected*
is True. The *is_public* field will be the only exception to the normal checks.


Alternatives
------------

Leave it unchanged

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

This may require a Horizon change (croberts please comment)

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  tmckay

Other contributors:
  croberts

Work Items
----------

Modify *is_protected* checks in the sahara engine
Modify unit tests
Horizon changes

Dependencies
============

None

Testing
=======

Unit tests

Documentation Impact
====================

None, unless there is a current section discussing protected/public

References
==========

None
