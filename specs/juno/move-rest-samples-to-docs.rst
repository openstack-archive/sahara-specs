..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==============================================
Move Sahara REST API samples from /etc to docs
==============================================


https://blueprints.launchpad.net/sahara/+spec/move-rest-samples-to-docs

Initially this idea was raised during discussions about
moving/releasing/versioning of Sahara's subprojects.
REST samples are slightly outdated and common documentation
is a good place to have them there to keep them up-to-date.


Problem description
===================

Today REST API samples are outdated and don't reflect current state of
changes made in Sahara api since Havana release. Also it's not obvious to
find those samples in Sahara sources. The goal is to update current state
of samples and move it to http://docs.openstack.org/


Proposed change
===============

Create a new page in docs::

  sahara/doc/source/restapi/rest_api_samples.rst

Move all JSON examples from sahara/etc/rest-api-samples to the new page.
Simple description for each example should be added before JSON code blocks.


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

Examples won't be found in the sahara/etc dir any longer.

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

Primary Assignee:
Alexander Ignatov

Work Items
----------

The following steps should be done:

* Move REST samples from sahara/etc to a new page in docs
* Update samples in docs according to current state of Sahara api
* Remove sahara/etc/rest-api-samples directory in Sahara sources

Dependencies
============

None


Testing
=======

None

Documentation Impact
====================

New page with information about REST samples will appear in the Sahara docs.


References
==========

None
