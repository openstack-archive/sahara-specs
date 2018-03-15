..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===========================
Plugins outside Sahara core
===========================

Plugins are a very important part of Sahara, they allow the creation of
clusters for different data processing tools and the execution of jobs on those
clusters.
This proposal is to remove the plugins code from Sahara core and create a new
project to host the plugins.

Problem description
===================

With the plugins code inside Sahara core we are limited to upgrade plugins
versions with the cycle milestone. It also forces the user to upgrade OpenStack
version whenever he/she needs to upgrade Sahara plugins.

Proposed change
===============

We are going to move the plugins to its own project, releasing new versions
when we upgrade new plugins, thus allowing the users to upgrade to newer
versions without the hussle of upgrading the whole cloud.

In order to keep the projects as less coupled as possible we are implementing a
mediator under sahara/plugins that will be used as an API between the projects.
Also this API aims to facilitate manutenability of both projects.

Alternatives
------------

Keep the plugins code as it is. Not changing will not break anything or make
things more difficult to the users.

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

User will be able to upgrade plugins versions a lot faster once this is done.
Deployer will have to keep an eye for the compatibility between sahara and
sahara-plugins if there is significant changes to the API.

There is also an impact for packagers and translators since we will need to do
one-time work to setup and copy translations in the new repository.

Developer impact
----------------

With a new project, developers will have to get used to the fact that plugins
don't live with the core anymore.
There is a new API (mediator) implemented on the sahara side that will be the
bridge between the two projects. Developers must respect that mediator and
significant changes to that will require version bumping or branching.

Image Packing impact
--------------------

Image packing using the new image generation and validation system will
require to have sahara-plugins installed as well.

Sahara-dashboard / Horizon impact
---------------------------------

None

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  tellesnobrega

Work Items
----------

* Split the plugins code from Sahara core
* Bring plugins unit test to the plugins repo
* Make sure imports in sahara-plugins from sahara are well structured so not to
  break with sahara changes later on

Dependencies
============

None

Testing
=======

Move plugins tests from sahara core to sahara-plugins

Documentation Impact
====================

We need to update the documentation to reflect the change and make sure users
and developers are well aware of this new structure.

References
==========

None

