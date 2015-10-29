..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==============================
Add CM API Library into Sahara
==============================

https://blueprints.launchpad.net/sahara/+spec/add-lib-subset-cm-api

This specification proposes to add CM API library into Sahara so that the
Cloudera plugin will not have to depend on a third party library support.

Problem description
===================

Now the Cloudera plugin is depending on a third party library cm_api (by
Cloudera). This library is not python3 compatible, and Cloudera has no
resource to fix it. Therefore, it is hard to put this library into
requirements.txt file. To fix this issue, we plan to implement a subset of
CM APIs and put them into Sahara project, so that the Cloudera plugin will not
depend on a third-party library, and we can enable this plugin by default.

Proposed change
===============

Now the CM APIs used in Cloudera plugin include (maybe not all included):

* ApiResource.create_cluster
* ApiResource.get_cluster
* ApiResource.get_all_hosts
* ApiResource.delete_host
* ApiResource.get_cloudera_manager
* ClouderaManager.create_mgmt_service
* ClouderaManager.hosts_start_roles
* ClouderaManager.get_service
* ApiCluster.start
* ApiCluster.remove_host
* ApiCluster.create_service
* ApiCluster.get_service
* ApiCluster.deploy_client_config
* ApiCluster.first_run
* ApiService.create_role
* ApiService.delete_role
* ApiService.refresh
* ApiService.deploy_client_config
* ApiService.start
* ApiService.restart
* ApiService.start_roles
* ApiService.format_hdfs
* ApiService.create_hdfs_tmp
* ApiService.create_yarn_job_history_dir
* ApiService.create_oozie_db
* ApiService.install_oozie_sharelib
* ApiService.create_hive_metastore_tables
* ApiService.create_hive_userdir
* ApiService.create_hive_warehouse
* ApiService.create_hbase_root
* ApiService.update_config
* ApiServiceSetupInfo.add_role_info
* ApiRole.update_config

Those APIs are what we need to implement in our CM APIs. We can create a
directory client in plugin cdh directory, and put the lib files in this
directory.

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

None

Deployer impact
---------------

Deployers will no longer need to install cm_api package anymore.

Developer impact
----------------

When new version of CDH is released, if it is incompatible with current used
client, or use some new APIs, developer may need to update the client when
adding support for new CDH release.

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
  ken chen

Other contributors:
  ken chen

Work Items
----------

The work items will include:

* Add a directory client in cdh plugin directory, and put lib files under
  this directory.
* Change all current cm_api imports into using the new client.

Dependencies
============

None

Testing
=======

Sahara Integration test for CDH plugin is enough.

Documentation Impact
====================

Documents about CDH plugin prerequisites and enabling should be modified, for
cm_api is not required any more.

References
==========

https://pypi.python.org/pypi/cm-api/8.0.0
