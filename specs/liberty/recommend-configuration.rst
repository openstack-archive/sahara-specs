..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=================================================================
Provide ability to configure most important configs automatically
=================================================================

https://blueprints.launchpad.net/sahara/+spec/recommend-configuration

Now users manually should configure most important hadoop configurations.
It would be friendly to provide advices about cluster configurations for
users.

Problem description
===================

Now users manually should configure most important hadoop configs, but it's
required to have advanced knowledge in Hadoop. Most configs are complicated
and not all users know them. We can
provide advices about cluster configuration and automatically configure
few basic configs, that will improve user experience. Created workaround
can extended in future with new confiuguration and advices.

Proposed change
===============

It's proposed to add calculator, which would automatically configure
most important configurations in dependency cluster specification:
available disk space, ram, cpu, and so on. Such calculator already
implemented in Ambari (see [1] and [2]), and we can use it as well. We should
have ability to switch off autoconfiguration and if user also manually
configured some hadoop config, autoconfiguration also will not be applied.

The following list of configs will be configured, using formulas from [1] and
[2]:

* yarn.nodemanager.resource.memory-mb
* yarn.scheduler.minimum-allocation-mb
* yarn.scheduler.maximum-allocation-mb
* yarn.app.mapreduce.am.resource.mb
* yarn.app.mapreduce.am.command-opts
* mapreduce.map.memory.mb
* mapreduce.reduce.memory.mb
* mapreduce.map.java.opts
* mapreduce.reduce.java.opts
* mapreduce.task.io.sort.mb

Also as a simple example we can autoconfigure before cluster validation
``dfs.replication`` if amout of ``datanodes`` less than default value.

Also it's required to add new plugin SPI method ``recommend_configs`` which
will autoconfigure cluster configs.

Alternatives
------------

None

Data model impact
-----------------

It's required to add new column ``use_autoconfig`` to cluster,
cluster_template, node_group, node_group_template, templates_relations
objects in DB. By default ``use_autoconfig`` will be ``True``. If
``use_autoconfig`` is ``False``, then we will not use autoconfiguration
during cluster creation. If none of the configs from the list above are
configured manually and ``use_autoconfig`` is ``True``, then we will
autoconfigure configs from list above. Same behaviour will be used for
node_groups configs autoconfiguration.

REST API impact
---------------

Need to support of switch off autoconfiguration.

Other end user impact
---------------------

Need to support of switch off autoconfiguration via python-saharaclient.

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

Need to add new checkbox which will allow to swith off autoconfiguration from
Horizon during cluster creation/scaling. If plugin doesn't support autoconfig
this checkbox will not be displayed. We can use ``_info`` field at [3] for
field.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  vgridnev

Other contributors:
  sreshetniak

Work Items
----------

Proposed change will consists with following steps:

* Implement new plugin SPI method which will provide configuration advices;
* Add support of this method in following plugins: CDH, Vanilla 2.6.0,
  Spark (``dfs.replication`` only);
* Provide ability to switch on autoconfiguration via UI;
* Provide ability to switch on autoconfiguration via saharaclient;
* Update WADL docs about new feilds objects.

Dependencies
============

Depends on Openstack requirements

Testing
=======

Unit tests will be implemented for this feature. Sahara CI also can start use
autoconfiguration as well.

Documentation Impact
====================

Need to document feature and all rules, which will be used for
autoconfiguration.

References
==========

[1] https://apache.googlesource.com/ambari/+/a940986517cbfeb2ef889f0d8a45579b27adad1c/ambari-server/src/main/resources/stacks/HDP/2.0.6/services/stack_advisor.py
[2] https://apache.googlesource.com/ambari/+/a940986517cbfeb2ef889f0d8a45579b27adad1c/ambari-server/src/main/resources/stacks/HDP/2.1/services/stack_advisor.py
[3] https://github.com/openstack/sahara/blob/master/sahara/service/api.py#L188
