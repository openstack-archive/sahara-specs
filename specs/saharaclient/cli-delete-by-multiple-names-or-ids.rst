..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

====================================
CLI: delete by multiple names or ids
====================================

https://blueprints.launchpad.net/python-saharaclient/+spec/cli-delete-by-multiple-names-or-ids


In sahara-cli you can only delete an object by name or id. In several
os clients, such as nova and heat, you can delete an object by
providing a list of ids or names. This blueprint is about adding this
feature in sahara-cli.




Problem description
===================

The CLI does not include deletion of objects by list of names or ids.

Proposed change
===============

* Long term solution

Our long term goal is to have the sahara-cli more consistent with
other os clients.

Current CLI usage for cluster-delete::

  sahara cluster-delete [--name NAME] [--id <cluster_id>]

Nova CLI usage for delete::

  nova delete <server> [<server> ...]

In nova-cli, and other os clients, you pass directly the id(s) or the
name(s) of the items you want to delete. We can refactor sahara-cli
to remove the --name and --id arguments. So in long term the usage
of sahara cli will be::

  sahara cluster-delete <cluster> [<cluster> ...]

  Positional arguments:
   <cluster>  Name or ID of cluster(s).``


* Short term solution

Note that the CLI refactoring will take substantial time, so as
short term solution,  can temporary use --names and --ids for all
delete verbs of the CLI. And once the CLI will be refactored,
we will remove all --name(s) and --id(s) arguments.

So the proposed change implies to add --names and --ids arguments
which consist of a Comma separated list of names and ids::

  sahara cluster-delete [--name NAME] [--id cluster_id]
                        [--names NAMES] [--ids IDS]


Alternatives
------------

No short term solution and just depend on the CLI refactoring
to provide this feature.


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

Update CLI methods \*_delete in saharaclient/api/shell.py


Assignee(s)
-----------

Primary Assignee:
Pierre Padrixe (stannie)

Work Items
----------

* Add delete by list of names or ids in the CLI
* Once the CLI is refactored, remove --name(s) --id(s) arguments


Dependencies
============

* For long term solution: we depend on the refactoring of the CLI
* For short term solution: none

Testing
=======

Update the tests to delete list of names and ids

Documentation Impact
====================

Documentation of the CLI needs to be updated


References
==========

None
