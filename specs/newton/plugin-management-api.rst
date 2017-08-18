..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==============================
Admin API for Managing Plugins
==============================

https://blueprints.launchpad.net/sahara/+spec/plugin-management-api

This is a proposal to implement new admin API for management plugins
for different projects. Also, smarter deprecation can be implemented
with a new management API for plugins.

Problem description
===================

Right now we have the following problems:

 * plugin state is totally unclear for user. From the user point
   of view, there are only default plugins, which are enabled for
   all projects;
 * deprecation of the particular plugin version is not perfect right
   now: validation error with deprecation note will be raised during
   creation - which is not so good for user;
 * all plugins and versions are enabled for all projects, which is
   not perfect. Someone wants to use only Ambari plugin, and other one
   wants to use only CDH plugin (or a particular version)

Proposed change
===============

It's proposed to implement several new API calls for plugins management.
Actually, there is plan to support that in API v1.1 correctly so that we
can put this to API v2 without additional stuff.

First of all we need to enable all plugins by default. There is no sense
to continue separating default and non-default plugins. There is an agreement
that having default/non-default plugins is redundant, just because such
process don't allow to deliver probably non-stable plugins at least for testing
in several projects.

Additionally as a part of this blueprint migration of ``hadoop_version``
to ``plugin_version`` should be done to keep this api consistent with new
management API.

New database should be created to store all required metadata about current
state of each plugin. This metadata is a combination of labels for plugins
itself and for each version of the plugin. The plugin label is an indicator
of plugin state (or version) that we will help user to understand the current
state of plugin itself (like a stability, deprecation status) or of some
features. If no metadata will be stored in DB, plugin SPI method will
help us to return default metadata for plugin. This also will help us
to avoid possible issues with upgrades from old releases of sahara.

This metadata should describe each plugin and its versions. The example
of return value of this plugin SPI method is the following:

.. sourcecode:: json

 {
    "version_labels": {
        "2.3": {
            "stable": {
                "status": true
            },
            "enabled": {
                "status": true
            }
        },
        "2.2": {
            "deprecated": {
                "status": true
            },
            "enabled": {
                "status": true
            }
        }
    },
    "plugin_labels": {
        "enabled": {
            "status": false,
        }
    }
 }
..

PluginManager on requests of all plugins will collect all data stored in DB
for plugins and will accumulate that data with default data for plugins without
entry in DB. Also, for each label entry manager will additionally put the
description of the label and possibility of changing of label by admins. The
collected data will be exposed to the user. See example of return value below
in REST API section.

The initial set of labels is the following:

 * ``enabled`` if plugin is enabled, then each user will be able to use plugin
   for cluster creation and will have ability to perform all CRUD operations on
   top the cluster. If plugins doesn't have this label, only deletion can be
   performed with this cluster.
 * ``hidden`` if plugin is hidden, it's still available for performing actions
   using saharaclient, but it will be hidden from UI side and CLI. It's special
   tag for fake plugin.
 * ``stable`` plugin is stable enough to be used. Sahara CI should be enabled
   for plugin to prove it's stability in terms of cluster creation and running
   EDP jobs. This label can't be removed. Will not
   be stored in DB and will be handled by plugin SPI method only.
 * ``deprecated`` plugin is deprecated. Warnings about deprecation
   will be shown for this plugin. Not intended to be used. Sahara CI (nightly)
   will continue testing this plugin to prove it's still works well. Label
   can't be removed. Recommendations should be provided about what operations
   are available for this cluster.

Admin user will be able to perform ``PATCH``  actions with labels via API,
if label is really can be removed. ``oslo_policy`` will be used for handling
that user have admin role. Only status can be changed for each label.
Mutability and description can't be changed.

Alternatives
------------

None

Data model impact
-----------------

New table is needed for this feature to store data about plugin tags.

.. sourcecode:: console

 +----------------+--------------+
 | plugins        | column type  |
 +----------------+--------------+
 | tenant_id      | String       |
 +----------------+--------------+
 | plugin_labels  | JsonDictType |
 +----------------+--------------+
 | version_labels | JsonDictType |
 +----------------+--------------+
 | id (Unique)    | String       |
 +----------------+--------------+
 | name           | String       |
 +----------------+--------------+


..

An simple example of stored data:

.. sourcecode:: console

 {
    'name': "fake",
    "plugin_labels": {
        "enabled": {
            "status": true,
        }
    },
    "tenant_id": "uuid",
    "id": "uuid just to be unique",
    "version_labels": {
        "0.1": {
            "enabled": {
                "status": true
            }
        }
    }
 }

..

REST API impact
---------------

There are bunch of changes in REST API are going to be done.

Endpoint changes:

1. for ``GET`` ``/plugins`` to following output will be expected after
   implementation. All labels will be additionally serialized with description,
   mutability.

.. sourcecode:: console

 {
    "plugins": [
        {
            "description": "HDP plugin with Ambari",
            "versions": [
                "2.3",
                "2.4",
            ],
            "name": "ambari",
            "plugin_labels": {
                "enabled": {
                    "description": "Indicates that plugin is switched on",
                    "mutable": true,
                    "status": true
                }
            },
            "version_labels": {
                "2.3": {
                    "enabled": {
                        "description": "Indicates that version is switched on",
                        "mutable": true,
                        "status": false,
                    },
                    "deprecated": {
                        "description": "Plugin is deprecated, but can be used"
                        "mutable": false,
                        "status": true
                    },
                    "stable": {
                        "description": "Plugin stability",
                        "mutable": false,
                        "status": false
                    }
                },
                "2.4": {
                    "enabled": {
                        ..
                    },
                    "stable": {
                        ..
                    },
                },
            },
            "title": "HDP Plugin"
        },
    ]
 }

..

2. new ``PATCH /plugins/<name>`` which is intended for updating tags for plugin
   or/and its versions. Update will be done successfully if all modified labels
   are mutable. Validation will be done for user if updating only
   status of each labels. To update a label you need to send request with
   only this label in body. Mutability and description are fields that can't be
   changed.

.. sourcecode:: console

 {
    "plugin_labels": {
        "enabled": {
            "status": false,
        }
    }
    "version_labels: {
        "2.3": {
            "enabled": {
                "status": true,
            },
        },
        "2.4": {
            "enabled": {
                "status": false,
            },
        },
    }
 }

..

Other end user impact
---------------------

New CLI will be extended with plugin updates. Warnings about
deprecation label will be added too.

Deployer impact
---------------

Nothing additional is required from deployers; anyway we should notify about
new default value for ``plugins`` option.

Developer impact
----------------

None

Sahara-image-elements impact
----------------------------

None

Sahara-dashboard / Horizon impact
---------------------------------

Things to do:

1. New tab for management plugins should be implemented. All labels
   will be shown in this tab. Each label will have checkboxes that will add
   this label to plugin. Only admin will have ability to produce changes.
2. Warning regarding deprecation label will be added to templates/cluster
   creation tabs. If the only plugin enabled we will not have dropdown for
   plugin choice, and the same thing for version. If the only plugin and
   version is enabled, plugin choice action will be skipped.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  vgridnev (Vitaly Gridnev)

Work Items
----------

The following items should be covered:

 * enable all plugins by default;
 * implement database side;
 * new API methods should be added;
 * plugin SPI method for default metadata;
 * document new api features in API docs;
 * python-saharaclient implementation;
 * sahara-dashboard changes

Dependencies
============

Depends on OpenStack requirements

Testing
=======

Feature will covered by unit tests.

Documentation Impact
====================

All plugin labels should be documented properly.

References
==========

None
