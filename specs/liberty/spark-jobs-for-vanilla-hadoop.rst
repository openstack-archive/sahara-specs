..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

====================================
Run Spark jobs on vanilla Hadoop 2.x
====================================

https://blueprints.launchpad.net/sahara/+spec/spark-jobs-for-vanilla-hadoop

This specification proposes to add ability to run Spark jobs on cluster
running vanilla version of Hadoop 2.x (YARN).

Problem description
===================

Support for running Spark jobs in stand-alone mode exists as well as for CDH
but not for vanilla version of Hadoop.

Proposed change
===============

Add a new edp_engine class in the vanilla v2.x plugin that extends
the SparkJobEngine. Leverage design and code from blueprint:
https://blueprints.launchpad.net/sahara/+spec/spark-jobs-for-cdh-5-3-0

Configure Spark to run on YARN by setting Spark's configuration
file (spark-env.sh) to point to Hadoop's configuration and deploying
that configuration file upon cluster creation.

Extend sahara-image-elements to support creating a vanilla image
with Spark binaries (vanilla+spark).

Alternatives
------------

Withouth these changes, the only way to run Spark along with Hadoop MapReduce
is to run on a CDH cluster.

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

Requires changes to sahara-image-elements to support building a vanilla 2.x
image with Spark binaries. New image type can be vanilla+spark.
Spark version can be fixed at Spark 1.3.1.

Sahara-dashboard / Horizon impact
---------------------------------

None

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  Michael Le (mvle)

Work Items
----------

New edp class for vanilla 2.x plugin.
sahara-image-elements vanilla+spark extension.
Unit test

Dependencies
============

Leveraging blueprint:
https://blueprints.launchpad.net/sahara/+spec/spark-jobs-for-cdh-5-3-0

Testing
=======

Unit tests to cover vanilla engine working with Spark.

Documentation Impact
====================

None

References
==========

None
