..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=========================================
Specification Repository Backlog Refactor
=========================================

https://blueprints.launchpad.net/sahara/+spec/add-backlog-to-specs

This specification proposes the refactoring of the `sahara-specs` repository
to inform a new workflow for backlogged features. This new workflow will
increase the visibility of specifications that have been approved but which
require implementation.


Problem description
===================

Currently the `sahara-specs` repository suggests a layout that creates
"implemented" and "approved" subdirectories for each release. This pattern
could create duplicates and/or empty directories as plans that have been
approved for previous releases but have not been completed are moved and
copied around as necessary.

Additionally, this pattern can increase the barrier for new contributors
looking for features to work on and the future direction of the project. By
having a single location for all backlogged specifications it will increase
visibility of items which require attention. And the release directories
can be devoted entirely to specifications implemented during those cycles.


Proposed change
===============

This change will add a backlog directory to the specs directory in the
repository, and refactor the juno directory to contain only the specs
implemented in juno.

It will also update the documentation to clearly indicate the usage of the
backlog directory and the status of the release directories. This
documentation will be part of the root readme file, a new document in the
backlog directory, and a reference in the main documentation.

This change is also proposing a workflow that goes along with the
repository changes. Namely that any work within a release that is either
predicted to not be staffed, or that is not started at the end of a
release should be moved to the backlog directory. This process should be
directed by the specification drafters as they will most likely be the
primary assignees for new work. In situations where the drafter of a
specification feels that there will be insufficient resources to create
an implementation then they should move an approved specification to the
backlog directory. This process should also be revisited at the end of a
release cycle to move all specifications that have not been assigned to the
backlog.


Alternatives
------------

Keep the directory structure the way it is currently. This does not improve
the status of the repository but is an option to consider. If the
directory structure is continued as currently configured then the release
directories will need to create additional structures for each cycle's
incomplete work.

Refactor each release directory to contain a backlog. This is very similar to
leaving things in their current state with the exception that it changes
the names in the directories. This change might add a small amount of
clarification but it is unlikely as the current names for "implemented" and
"approved" are relatively self explanatory.


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

The main impact will be for crafters of specifications to be more aware of
the resource requirements to implement the proposed changes. And potentially
move their specifications based on those requirements.


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
  mimccune (Michael McCune)


Work Items
----------

* Create the backlog directory and documentation.
* Clean up the juno directory.
* Add references to backlog in the contributing documentation.


Dependencies
============

None


Testing
=======

None


Documentation Impact
====================

The "How to Participate" document should be updated to make reference to
the backlog directory as a place to look for new work.


References
==========

Keystone is an example of another project using this format for backlogged
work. Examples of the documentation for the backlog directory[1] and the
root documentation[2] will be used as reference in the creation of Sahara
specific versions.

[1]: https://github.com/openstack/keystone-specs/tree/master/specs/backlog
[2]: https://github.com/openstack/keystone-specs
