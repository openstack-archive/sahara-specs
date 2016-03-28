..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

======================================
Refactor the sahara.service.api module
======================================

https://blueprints.launchpad.net/sahara/+spec/v2-api-experimental-impl

The HTTP API calls that sahara receives are processed by the
``sahara.api`` package, the functions of this module then call upon the
``sahara.service.api`` and ``sahara.service.edp.api`` modules to perform
processing before being passed to the conductor. To help accommodate future
API changes, these modules should be refactored to create a more unified
package for future implementors.


Problem description
===================

The current state of the API service level modules can be confusing when
comparing it to the API route level modules. This confusion can lead to
misunderstandings in the way the service code interacts with the routing
and conductor modules. To improve the state of readability, and expansion,
this spec defines a new layout for these modules.

More than confusion, as the API is being reworked for the new version 2
features there will need to be additions in the service layer to assist
the endpoint and JSON notational changes. Refactoring these modules will
create a more clear pathway for adding to the sahara API.

Proposed change
===============

This change will create a new package named ``sahara.service.api`` which will
contain all the service level API modules. This new package will create a
unified location for service level API changes, and provide a clear path for
those wishing to make said changes. The new package will also contain the
base service level files for the v2 API.

The new package layout will be as follows:

::

    sahara/service/api/__init__.py
                       v10.py
                       v11.py
                       v2/__init__.py
                          clusters.py
                          cluster_templates.py
                          data_sources.py
                          images.py
                          job_binaries.py
                          job_executions.py
                          jobs.py
                          job_types.py
                          node_group_templates.py
                          plugins.py


This new layout will provide a clean, singular, location for all service
level API code. The files created for the ``sahara.service.api.v2`` package
will be simple copies of the current functionality.

Alternatives
------------

One alternative is to do nothing and leave the ``sahara.service.api`` and
``sahara.service.edp.api`` modules as they are and create new code for v2
either in those locations or a new file. A downside to this approach is that
it will be less clear where the boundaries between the different versions
will exist. This will also leave a larger questions as to where the new v2
code will live.

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

This change should improve the developer experience with regard to creating
and maintaining the API code as it will be more clear which modules control
each version.

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
    Michael McCune (elmiko)

Work Items
----------

* create new package
* move current implementations into new package
* add v2 related service files
* fix outstanding references
* add documentation to v2 developer docs


Dependencies
============

None


Testing
=======

As these changes will leave the interfaces largely intact, they will be
tested through our current unit and tempest tests. The will not require
new tests specifically for this change.


Documentation Impact
====================

A new section in the API v2 developer docs will be added to help inform about
the purpose of these files and how they will be used in the work to
implement features like JSON payload changes.


References
==========

None
