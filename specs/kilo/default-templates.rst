..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=================================
Default templates for each plugin
=================================

Blueprint:  https://blueprints.launchpad.net/sahara/+spec/default-templates

In order to create a basic cluster for any plugin, a user currently has to
go through several steps to create an appropriate set of node group and
cluster templates.  We believe that set of actions should be captured in a
default set of templates (per plugin) and loaded into the database ahead of
time so that users do not need to repeat some of the most basic steps to
provision a simple cluster.

Problem description
===================

Creating even a basic cluster currently requires several steps before the
user is able to launch their cluster.  In an effort to reduce the amount of
effort and time to get a simple cluster running, we propose a set of default
templates for each plugin that will be pre-loaded and available for use.

Other potential issues/answers:
1) How do we make the set of templates available for all users/tenants?
Today, any given template is only read/writable by one tenant.

A) Implement ACL support for templates (SLukjanov plans to write the spec
for this).  Having proper ACL support in place will allow us to manage
access for read/write across all tenants.

2) Do we allow editing of default templates?
I don't think we should allow editing of the default templates since they
are shared among all tenants.  The flow to "edit" one would be to make a
copy of the template and work from there.  I propose that each template will
be stored with a flag in the database that identifies each default template
as being a default template so that we can enforce that users cannot change
the default templates.

3) How do we avoid uploading the same default templates each time at startup
while still allowing for them to be updated as necessary?
We could use a numbering system in the template file names to indicate the
version number and store that in the database (perhaps instead of a boolean
flag indicating that the template is a default template,
we could store an int that is the version number).  At startup time,
we would go through all of the template files for each plugin and compare
the version numbers to the version numbers that are stored in the database.

A)  1) Sahara starts up (or "something" starts up), looks in the
sahara/plugins/version/resources dirs for templates 2) Read each template,
find the name 3) Look at the modification time on the file 4) Look up the
template in the db by name and check the "updated" field. If the updated field
is later than the modification time, leave it alone otherwise update it.
If the record can't be found by name, upload it.

4) How do we make a json default cluster template reference a json default node
group template since we won't know the node group template IDs?

A) CLI util will operate them one-by-one starting with node group templates and
then cluster templates. In addition, we could create a few example jobs that
could be used for health check.

Proposed change
===============

1) Create a set of default template json files for each plugin.  I propose
that they will exist in sahara/plugins/<plugin>/<version>/resources.  The
contents of each default template set will be 1 or more node group templates
and a cluster template.

2) Add a CLI util that could be executed by cron with admin credentials to
create/update existing default templates.  This utility needs to be able to
take some placeholders like "flavor" or "network" and make the appropriate
substitutions (either from configs or via commnad line args) at runtime.
The cron job could be optional if we wanted to force any updates to be
triggered explicitly.

Alternatives
------------

1) The loading process could be done via the REST API if we wanted to have
some sort of external process that manages the default templates.  That might
require changing the API a bit to add endpoints for managing the default
templates and seems like a fair bit of unnecessary work since the management of
default templates should be something done only within Sahara.

2) Add a hook, possibly in plugins/base:PluginManager for
"load_default_templates".  This method would be responsible for triggering
the loading of the defaults at startup time.

Data model impact
-----------------

N/A

REST API impact
---------------

N/A

Other end user impact
---------------------

End users will see the default templates show up just like any other
template that they may have created.

Deployer impact
---------------

N/A

Developer impact
----------------

N/A

Sahara-image-elements impact
----------------------------

N/A

Sahara-dashboard / Horizon impact
---------------------------------

N/A
The default templates will show-up in the UI and look like regular templates.


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  croberts

Secondary assignee:
  tmckay

Work Items
----------

1) Come up with a set of default templates for each plugin.  These will
probably be json formatted files.

2) Come up with some sort of mechanism to load the templates or ensure that
they are already loaded when Sahara starts-up.

3) Update the Sahara documentation.

Dependencies
============

1)  Implementation of the ACL for templates (spec still TBD).  This will let
us give all users read access to the default templates while still possibly
allowing admins to edit the templates.

Testing
=======

Ideally, tests will be added to ensure that a functioning cluster can be
started based on each of the default template sets.  If that is determined
to be too time-consuming per-run, then tests to ensure the validity of each set
of templates may be sufficient.

Documentation Impact
====================

The Sahara documentation should be updated to note that the default
templates are available for use.  Additionally, any future plugins will be
expected to provide their own set of default templates.

References
==========

N/A
