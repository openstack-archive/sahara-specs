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

ACL support is not available yet. For the time being, templates will be
added per-tenant.

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

Sqlalchemy will detect when fields have changed and set the "updated at" field
accordingly.  Therefore, we can simply attempt to update existing templates
whenever the script is run. If the template matches what is in the database,
there will be no update.

4) How do we make a json default cluster template reference a json default node
group template since we won't know the node group template IDs?

A) CLI util will operate them one-by-one starting with node group templates and
then cluster templates. In addition, we could create a few example jobs that
could be used for health check.

Proposed change
===============

1) Create sets of default template json files for each plugin.

   A set of default templates will consist of node group templates and/or
   cluster templates. Whether or not a template file defines a node group
   template or a cluster template will be determined based on the presence of
   required fields specified in the Sahara JSON validation code. Node group
   templates require `flavor_id` and `node_processes`, so presence of these
   fields implicitly identify a template as a node group template.  If it is
   not a node group template, it is a cluster template.  This identification
   avoids a naming scheme or some other kind of extra labeling to identify a
   template type.

   Default templates distributed with Sahara will be located in
   `sahara/plugins/default_templates/`. The code that processes the default
   templates should use this path as a default starting point but should allow
   a different starting point to be specified. It should make no assumptions
   about the directory structure beyond the following:

   *  All of the template files in a particular directory should be treated as
      a set, and cluster templates may reference node group templates in the
      same set by name.

   * Directories may be nested for logical organization, so that a plugin may
     define default template sets for each version. Therefore, the code should
     recurse through subdirectories by default but it should be possible to
     specify no recursion.

   This design will allow code to process default templates decoupled from
   explicit knowledge of Sahara plugins, plugin versions, or the structure of
   plugin directories. The ability to specify a different starting point
   will allow a user to process particular template sets if the entire set
   is not desired (only some plugins enabled, for instance), or if an alternate
   set at a different location is desired. In short, it keeps the processing
   general and simple.

   In practice, the directories under `sahara/plugins/default_templates` will
   be named for plugins, and subdirectories will be created for different
   versions as appropriate.


2) Add a CLI util that can be executed by cron with admin credentials to
   create/update existing default templates. This utility needs to be able to
   take some placeholders like "flavor" or "network" and make the appropriate
   substitutions (either from configs or via commnad line args) at runtime.
   The cron job can be optional if we want to force any updates to be
   triggered explicitly.

   The CLI will take an option to specify the starting directory (default
   will be `sahara/plugins/default_templates`).

   The CLI will take an option to disable recursion through subdirectories
   (default will be to recurse).

   At a minimum, the CLI will provide a command to create or update default
   templates with processing beginning at the designated starting directory.

   The CLI should use the "plugins" configuration parameter from the [database]
   section to filter the templates that are processed. If the "plugin-name"
   field of a template matches a plugin name in the "plugins" list, it will
   be processed.  If the "plugins" list is empty, all templates will be
   processed.  It should be possible to override the "plugins" configuration
   from the command line with a "--plugin-name" option.

   If there is an error during updating or creating templates in a particular
   set, the CLI should attempt to undo any modifications or creations that
   were done as part of that set.

   The CLI should also provide a mechanism for deleting default templates,
   since the `is_default` field will prevent that, should an admin for
   some reason desire to remove default templates. This can be a simple
   operation that will remove a single default template by ID. It is not
   likely to be used very often.

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
