..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=============================================
SaharaClient CLI as an OpenstackClient plugin
=============================================

https://blueprints.launchpad.net/python-saharaclient/+spec/cli-as-openstackclient-plugin

This specification proposes to create SaharaClient CLI as an OpenstackClient
plugin.

Problem description
===================

Currently SaharaClient CLI has a lot of problems and is not so attractive
as wanted to be. It should be refactored or recreated from the start.

Proposed change
===============

New SaharaClient CLI will be based on OpenstackClient that brings the command
set for different projects APIs together in a single shell with a uniform
command structure.

OpenStackClient provides first class support for the several services. The
other ones (including Data Processing service) may create an OpenStackClient
plugin.

Proposed Objects:

 * plugin
 * image
 * data source
 * job template
 * job
 * job binary
 * node group template
 * cluster template
 * cluster

Proposed Commands:

All commands will have a prefix ``dataprocessing``.
Arguments in ``[]`` are optional, in ``<>`` are positional.

For Plugins:

.. sourcecode:: shell

    plugin list [--long]
    plugin show <plugin>
    plugin configs get <plugin> <version> [--file <filepath>] # default name of
    # the file is <plugin>

Detailed description of a plugin contains too much information to display it
on the screen. It will be saved in file instead. If provided file exists, data
will not be rewritten.

Columns for ``plugin list``: name, versions.
Columns for ``plugin list --long``: name, versions, title, description.
Rows for ``plugin show``: name, versions, title, description.

For Images:

.. sourcecode:: shell

    image list [--tags <tag(s)>] [--long]
    image show <image>
    image register <image> <username> [--description <description>]
    image unregister <image(s)>
    image tags set <image> <tag(s)>
    image tags add <image> <tag(s)>
    image tags remove <image> <tag(s)>

``tags set`` will replace current image tags with provided ones.
``tags remove`` will support passing ``all`` to ``tags`` argument to remove all
tags.

Columns for ``image list``: name, id, username, tags.
Columns for ``image list --long``: name, id, username, tags, status,
description.
Rows for ``plugin show``: name, id, username, tags, status, description.

For Data Sources:

.. sourcecode:: shell

    data source create <name> <type> <url> [--password <password>]
        [--username <user>] [--description <description>]
    data source list [--type <type>] [--long]
    data source show <datasource>
    data source delete <datasource(s)>
    data source update <datasource> [--name <name>] [--type <type>]
        [--url <url>] [--password <password>] [--username <user>]
        [--description <description>]

Columns for ``data source list``: name, id, type.
Columns for ``data source list --long``: name, id, type, url, description.
Rows for ``data source show``: name, id, type, url, description.

New CLI behavior in case of Node Group Templates, Cluster Templates and
Clusters creation will be pretty much the same as in Horizon, but additionally
will allow to create them from json.
It doesn't let to mark some arguments as required for successful creation, but
it could be done in help strings.

For Job Binaries:
Job Binaries and Job Binary Internals will be combined

.. sourcecode:: shell

    job binary create <name> [--data <filepath>] [--description <description>]
        [--url <url>] [--username <username>] [--password <password>]
    job binary list [--name <name-regex>]
    job binary show <job-binary>
    job binary update <job-binary> [--description <description>] [--url <url>]
        [--username <username>] [--password <password>]
    job binary delete <job-binary(ies)>
    job binary download <job-binary> [--file <filepath>]

Columns for ``job binary list``: name, id.
Columns for ``job binary list --long``: name, id, url, description.
Rows for ``job binary show``: name, id, url, description.

For Node Group Templates:

.. sourcecode:: shell

    node group template create [--name <name>] [--plugin <plugin>]
        [--version <version>] [--flavor <flavor>] [--autoconfigs]
        [--node-processes <node-processes>] [--floating-ip-pool <pool>]
        [--proxy-gateway] [--configs <filepath>] [--json <filepath>]
        # and other arguments except of "image-id"
    node group template list [--plugin <plugin>] [--version <version>]
        [--name <name-regex>] [--long]
    node group template show <node-group-template>
    node group template configs get <node-group-template> [--file <filepath>]
        # default name of the file is <node-group-template>
    node group template update <node-group-template> ... [--json <filepath>]
        # and other arguments the same as in create command
    node group template delete <node-group-template(s)>

Columns for ``node group template list``: name, id, plugin, version.
Columns for ``node group template list --long``: name, id, plugin, version,
node-processes, description.
Rows for ``node group template show``: name, id, plugin, version,
node-processes, availability zone, flavor, is default, is proxy gateway,
security groups or auto security group, if node group template contains
volumes following rows will appear: volumes per node,
volumes local to instance, volumes mount prefix, volumes type,
volumes availability zone, volumes size, description.

For Cluster Templates:

.. sourcecode:: shell

    cluster template create [--name <name>] [--description <description>]
        [--node-groups <ng1:1,ng2:2>] [--anti-affinity <node-processes>]
        [--autoconfigs] [--configs <filepath>] [--json <filepath>]
    cluster template list [--plugin <plugin>] [--version <version>]
        [--name <name-regex>] [--long]
    cluster template configs get <cluster-template> [--file <filepath>]
        # default name of the file is <cluster-template>
    cluster template show <cluster-template>
    cluster template update <cluster-template> ... [--json <filepath>]
        # and other arguments the same as in create command
    cluster template delete <cluster-template(s)>

Plugin and its version will be taken from node group templates.

Columns for ``cluster template list``: name, id, plugin, version.
Columns for ``cluster template list --long``: name, id, plugin, version,
node groups (in format name:count), description.
Rows for ``cluster template show``: name, id, plugin, version,
node groups, anti affinity, description.

For Clusters:

.. sourcecode:: shell

    cluster create [--name <name>] [--cluster-template <cluster-template>]
        [--description <description>][--user-keypair <keypair>]
        [--image <image>] [--management-network <network>] [--json <filepath>]
        [--wait]
    cluster scale [] [--wait]
    cluster list [--plugin <plugin>] [--version <version>]
        [--name <name-regex>] [--long]
    cluster show <cluster>
    cluster delete <cluster(s)> [--wait]

If ``[--wait]`` attribute is set, CLI will wait for command completion.
Plugin and its version will be taken from cluster template.

Columns for ``cluster list``: name, id, status.
Columns for ``cluster list --long``: name, id, url, description.
Rows for ``cluster show``: name, id, anti affinity, image id, plugin, version,
is transient, status, status_description, user keypair id, description.

For Job Templates (Jobs):

.. sourcecode:: shell

    job template create [--name <name>] [--type <type>]
        [--main-binary(ies) <mains>] [--libs <libs>] [--description <descr>]
        [--interface <filepath>] [--json <filepath>]
    job template list [--type <type>] [--name <name-regex>] [--long]
    job template show <job-template>
    job template delete <job-template>
    job template configs get <type> [--file <file>] # default file name <type>
    job types list [--plugin <plugin>] [--version <version>] [--type <type>]
        [--hints] [--file <filepath>] # default file name depends on provided
        # args

``job types list`` and ``job template configs get`` outputs will be saved in
file just like ``plugin configs get``.

Columns for ``job template list``: name, id, type.
Columns for ``job template list --long``: name, id, type, libs(ids),
mains(ids), description.
Rows for ``job template show``: name, id, type, libs(ids),
mains(ids), description.

For Jobs (Job Executions):

.. sourcecode:: shell

    job execute [--job-template <job-template>] [--cluster <cluster>]
        [--input <data-source>] [--output <data-source>] [--args <arg(s)>]
        [--params <name1:value1,name2:value2>]
        [--configs <name1:value1,name2:value2>]
        [--interface <filepath>] [--json <filepath>] [--wait]
    job list [--long]
    job show <job>
    job delete <job(s)> [--wait]

Columns for ``job list``: id, cluster id, job id, status.
Columns for ``job list --long``: id, cluster id, job id, status, start time,
end time
Rows for ``job show``: id, cluster id, job id, status, start time,
end time, input id, output id

If ``[--wait]`` attribute is set, CLI will wait for command completion.

Besides this, there are a bunch of arguments provided by OpenstackClient, that
depends on chosen command plugin.
For example, there is a help output for ``plugin list`` command:

.. sourcecode:: shell

    (openstack) help dataprocessing plugin list
    usage: dataprocessing plugin list [-h] [-f {csv,html,json,table,value,
                                                                    yaml}]
                                  [-c COLUMN] [--max-width <integer>]
                                  [--quote {all,minimal,none,nonnumeric}]
                                  [--long]

    Lists plugins

    optional arguments:
    -h, --help            show this help message and exit
    --long                List additional fields in output

    output formatters:
    output formatter options

      -f {csv,html,json,table,value,yaml}, --format {csv,html,json,table,value,
                                                                          yaml}
                        the output format, defaults to table
      -c COLUMN, --column COLUMN
                            specify the column(s) to include, can be repeated

    table formatter:
      --max-width <integer>
                            Maximum display width, 0 to disable

    CSV Formatter:
      --quote {all,minimal,none,nonnumeric}
                            when to include quotes, defaults to nonnumeric

Alternatives
------------

Current CLI code can be refactored.

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

* Creating OpenstackClient plugin for SaharaClient
* Commands implementation for each object, described in "Proposed change"
  section
* Updating documentation with corresponding changes
* Old CLI will be deprecated and removed after some time

Dependencies
============

None

Testing
=======

Every command will be provided with unit tests.

Documentation Impact
====================

Documentation about new CLI usage will be written.

References
==========

`OpenstackClient documentation about using Plugins <http://docs.openstack.org/developer/python-openstackclient/plugins.html>`_
`OpenstackClient documentation about Objects and Actions naming <http://docs.openstack.org/developer/python-openstackclient/commands.html>`_
