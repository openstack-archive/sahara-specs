..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

============================
Initial Kerberos Integration
============================

https://blueprints.launchpad.net/sahara/+spec/initial-kerberos-integration

There is no way to enable kerberos integration for clusters deployed
by Sahara. By the way, default managament services like Cloudera Manager
and Ambari Management console already have support of configuring Kerberos
for deployed clusters. We should make initial integration of this feature in
sahara for Cloudera and Ambari plugins.

Problem description
===================

Users wants to use Kerberos to make clusters created by Sahara more
secure.

Proposed change
===============

The proposed changes will be the following:

 * the initial MIT of KDC integration on cluster nodes. Also
   admin principal will be created for sahara usages (and
   sahara will store that using improved secret storage);
 * ability to configure Kerberos on clusters created by using
   Cloudera Manager API (see the reference below [0], [1], [2]) and
   Ambari (see the reference below [3]) will be added;
 * remote operations which are requires auth should be wrapped by
   ticket granting method so that sahara will be able to perform
   operations with hdfs and something like that;
 * Oozie client should be re-implemented to reflect these changes.
   By default, in case when cluster is not deployed with Kerberos
   (like vanilla) we will continue using requests python library
   to bind Oozie API (without needed auth). The new Oozie client
   should be implemented in case of kerberos implemented. This
   client will use standard remote implementation and curl in order
   to process auth to Oozie with ticket granting. As another way, we can use
   request-kerberos for that goal, but it's not good solution since this lib
   are not python3 compatible.

New config options will be added in general section of cluster template (or
cluster):

 * ``Enable Kerberos``: will enable the kerberos security for cluster
   (Ambari or CDH);

As additional enhancement support of using existing KDC server can be added.
In such case, additional options are required for that:

 * ``Use existing KDC``: will enable using existing KDC server, additional
   data should be provided then;
 * ``Admin principal``: admin principal to have ability to create new
   principals;
 * ``Admin password``: will be hidden from API outputs and will be stored in
   improved secret storage;
 * ``KDC server hostname``: hostname of KDC server

If something additional will be needed to identify KDC server it also will
be added to general section of configs.

Other possible improvements can be done after implementation of steps above.
By the way, initial implementation will only include steps above.

Alternatives
------------

None

Data model impact
-----------------

None. If needed, extra field will be used for that goal.

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

All packages required for KDC infrastucture should be included on our
images. If something is not presented, it will be additionally installed.

Sahara-dashboard / Horizon impact
---------------------------------

Nothing additional is required on Sahara dashboard side, since all needed
options are in general section and will be prepared as usual

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  vgridnev (Vitaly Gridnev) and msionkin (Michael Ionkin)

Work Items
----------

All working items are will described in Proposed change. Additionally,
we can include testing initial Kerberos integration on Sahara CI, but only
when all steps are completed. For sure, unit tests will be added.

Dependencies
============

Depends on OpenStack requirements. As initial idea, nothing additional
should not be added to current sahara requirements, all needed packages
will be included only on sahara images.

Testing
=======

Sahara CI will cover that change, unit tests for sure will be added.

Documentation Impact
====================

New sections can be added with description of Kerberos integration
in Ambari and Cloudera.

References
==========

[0] https://github.com/cloudera/cm_api/blob/f4431606a690d95208457a64d1cc2610d9cfa2bf/python/src/cm_api/endpoints/cms.py#L134
[1] https://github.com/cloudera/cm_api/blob/f4431606a690d95208457a64d1cc2610d9cfa2bf/python/src/cm_api/endpoints/clusters.py#L585
[2] http://www.cloudera.com/documentation/enterprise/5-5-x/topics/cm_sg_using_cm_sec_config.html
[3] https://cwiki.apache.org/confluence/display/AMBARI/Automated+Kerberizaton#AutomatedKerberizaton-TheRESTAPI

