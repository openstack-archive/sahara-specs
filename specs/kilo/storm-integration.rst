..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=================
Storm Integration
=================


https://blueprints.launchpad.net/sahara/+spec/storm-integration

This blueprint aims to implement Storm as a processing option in Sahara.
Storm is a real time processing framework that is highly used in the big
data processing community.

Problem description
===================

Sahara today is only able to process big data in batch. Storm provides a
easy setup stream processing. Having storm integrated with Sahara gives sahara
a new feature, so not only users will be able to process batch but also real
time data.

Proposed change
===============

* The implementation is divided in three steps:

  * First we need to implement Storm Plugin.

    * Identify Storm configuration files and manage their creation via sahara;
    * Create the plugin itself to manage Storm deploy;

  * Second, we need to create a new Job Manager for storm, following Trevor
    McKay's refactoring
  * Last we will create a new image that comes with storm installed.

* Node Groups:

  * Storm has two basic components **Nimbus** and **Supervisor**, Nimbus is
    the master node, hosts the UI and is the main node of the topology.
    Supervisors are the worker nodes. Other than that, storm needs only
    zookeeper to run, we need have it as well.
    The basic Node Groups that we will have are:

      * Nimbus;
      * Supervisor;
      * Nimbus and Supervisor;
      * Zookeeper;
      * Nimbus and Zookeeper;
      * Supervisor and Zookeeper;
      * Nimbus, Supervisor and Zookeeper


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

None

Developer impact
----------------

None

Sahara-image-elements impact
----------------------------

Sahara-image-elements may be the place to create storm image, it will need
deeper investigation but looks like the best place to create and publish
storm images.

Sahara-dashboard / Horizon impact
---------------------------------

The first changes needed here is to show the configuration options of Storm
when this type of job is selected.
There will be also necessary to have a deploy job where the user can submit
the topology jar and the command line to run it.
As for now, I don't see other changes in the UI, the jobs are very simple and
need no special configurations.

Implementation
==============

Assignee(s)
-----------

Primary assignee:

  * tellesmvn

Work Items
----------

The implementation is divided in three steps:

* First we need to implement Storm Plugin.

  * Storm plugin is similar to Spark plugin. The implementation will be based
    on Spark's but necessary changes will be made.
  * Storm doesn't rely on many configuration files, there is only one needed
    and it is used by all nodes. This configuration file is written in YAML
    and it should be dinamically written in the plugin since it needs to have
    the name or ip of the master node and also zookeeper node(s). We will need
    PYYAML to parse this configuration to YAML. PYYAML is already a global
    requirement of OpenStack and will be added to Sahara's requirement as well.
  * The plugin will run the following processes:

    * Storm Nimbus;
    * Storm Supervisor;
    * Zookeeper.

* Second, we need to create a new Job Manager for storm, following Trevor
  McKay's refactoring

  * This implementation is under review and the details can be seen here:
    https://review.openstack.org/#/c/100678/

* Last we will create a new image that comes with storm installed.

  * This part is yet not the final decision, but it seems to me that it is
    better to have a prepared image with storm than having to install it
    every time a new cluster is set up.

Dependencies
============

* https://review.openstack.org/#/c/100622/

Testing
=======

I will write Unit Tests, basically to test the write configuration file and
more tests can be added as needed.

Documentation Impact
====================

We will need to add Storm as a new plugin in the documentation and write how
to use the plugin.
Also a example in sahara-extra on how to run a storm job will be provided

References
==========

* `Wiki <https://wiki.openstack.org/wiki/HierarchicalMultitenancy>`
* `Etherpad <https://etherpad.openstack.org/p/juno-summit-sahara-edp>`
* `Storm Documentation <http://storm.incubator.apache.org/>`
