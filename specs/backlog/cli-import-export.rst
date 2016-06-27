..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=====================================================
Adding ability to export and import templates via CLI
=====================================================

https://blueprints.launchpad.net/python-saharaclient/+spec/cli-import-export

This specification proposes to add an ability to export and import templates
via CLI.

Problem description
===================

It could be useful to move node group, cluster and job templates across
environments especially when they have a lot of configuration options.

Proposed change
===============

Optional argument ``--export`` will be added to
``dataprocessing {resource} show`` command. If this argument is provided,
``show`` command will expose preprocessed JSON to stdout (by default, if
filepath was not provided ``--export``) or to the file
(``--export <filepath>``).

This JSON will consists of fields, that can be used for resource creation.
If some field has an uuid as a value, these uuid will be replaced with
placeholder ``FIELD_NAME_ID``. Empty fields will not be printed.

All ``--json`` arguments will be replaced with ``--import`` arguments.
When JSON file with resource template is created, resource can be created from
it with ``dataprocessing {resource} create --import <json-file>`` command.
All uuids in initial JSON can be manually replaced with appropriate values or
added with arguments.

Example:

Export of node group template:

.. sourcecode:: console

    dataprocessing node group template show --export example.json

File example.json will have:

.. sourcecode:: json

    {
        "node_processes": [
            "namenode",
            "resourcemanager",
            "historyserver",
            "oozie"
        ],
        "name": "test",
        "plugin_name": "vanilla",
        "floating_ip_pool": "FLOATING_IP_POOL_ID",
        "is_default": false,
        "is_protected": false,
        "node_configs": {
            "HDFS": {},
            "Hadoop": {},
            "Hive": {},
            "MapReduce": {},
            "JobFlow": {},
            "YARN": {}
        },
        "use_autoconfig": true,
        "is_proxy_gateway": false,
        "is_public": false,
        "hadoop_version": "2.7.1",
        "auto_security_group": true,
        "security_groups": [],
        "flavor_id": "2"
    }

This node group can be created in some other environment:

.. sourcecode:: console

    dataprocessing node group template create --import example.json
        --floating-ip-pool 1234

JSON provided via ``--import`` will be considered as a base template and all
provided parameters will override it. For consistency maintaining, this
approach will be implemented for all other resources (not only for templates).

Alternatives
------------

Use templating format like Jinja2 and make tool for compiling it.

Data model impact
-----------------

None

REST API impact
---------------

None

Other end user impact
---------------------

New feature in CLI.

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

The ability to import/export can be implemented for sahara-dashboard as well,
but a separate spec should be written to describe these feature.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  apavlov-n

Work Items
----------

* Adding ``--export`` argument to ``show`` methods of node group, cluster and
  job templates;
* Changing work of ``--import`` argument for all resources;
* Adding documentation about exporting and importing templates.

Dependencies
============

None

Testing
=======

Will be covered with unit tests.

Documentation Impact
====================

Will be documented.

References
==========

None
