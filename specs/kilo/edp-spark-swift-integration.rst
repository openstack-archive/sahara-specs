..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=====================================
Enable Spark jobs to access Swift URL
=====================================

https://blueprints.launchpad.net/sahara/+spec/edp-spark-swift-integration

Spark uses Hadoop filesystem libraries to dereference input and output URLs.
Consequently, Spark can access Swift filesystem URLs if the Hadoop Swift JAR
is included in the image and a Spark job’s Hadoop configuration includes the
necessary Swift credentials. This specification describes a method of
transparently adding Swift credentials to the Hadoop configuration of a Spark
job so that the job source code does not need to be altered and recompiled to
access Swift URLs.

Problem description
===================

Spark jobs may access Swift URLs assuming the cluster image includes the
Hadoop Swift JAR. To do this, the job’s Hadoop configuration must include the
necessary Swift credentials (`fs.swift.service.sahara.username` and
`fs.swift.service.sahara.password`, for example).

As with Oozie Java actions, job source code may be modified and recompiled to
add the necessary configuration values to the job’s Hadoop configuration.
However, this means that a Spark job which runs successfully with HDFS input
and output sources cannot be used “as is” with Swift input and output sources.

Sahara should allow users to run Spark jobs with Swift input and output
sources without altering job source code.

Proposed change
===============

This change follows the approach developed by Kazuki Oikawa in
https://blueprints.launchpad.net/sahara/+spec/edp-improve-compatibility for
Java compatibility.

A new configuration value will be added to Sahara, `edp.spark.adapt_for_swift`.
If this configuration value is set to True on a Spark job, Sahara will run a
wrapper class (SparkWrapper) instead of the original class indicated by the job.
The default for this configuration value will be False.

Sahara will generate a `spark.xml` file containing the necessary Swift
credentials as Hadoop configuration values. This XML file will be uploaded to
the master node with the JAR containing the SparkWrapper class, along with the
other files normally needed to execute a Spark job.

Sahara’s Spark launcher script will run the SparkWrapper class instead of the
job’s designated main class. The launcher will pass the name of the XML
configuration file to SparkWrapper at runtime, followed by the name of the
original class and any job arguments. SparkWrapper will add this XML file to the
default Hadoop resource list in the job’s configuration before invoking the
original class with any arguments.

When the job’s main class is run, it’s default Hadoop configuration will
contain the specified Swift credentials.

The permissions of the job dir on the master node will be set to 0x700. This
will prevent users other than the job dir owner from reading the Swift values
from the configuration file.

The sources for the SparkWrapper class will be located in the
sahara-extra repository under the `edp-adapt-for-spark` directory.
This directory will contain a `pom.xml` so that the JAR may be built
with maven. Maintenance should be light since the SparkWrapper class is
so simple; it is not expected to change unless the Hadoop Configuration class
itself changes.

Currently, the plan is to build the JAR as needed and release it with
Sahara in `service/edp/resources/edp-spark-wrapper.jar`. Alternatively, the
JAR could be hosted publically and added to Sahara images as an element.

Alternatives
------------

There is no known way to supply Swift credentials to the Hadoop filesystem
libraries currently other than through configuration values. Additionally,
there is no way to add values to the Hadoop configuration transparently other
than through configuration files.

This does present some security risk, but it is no greater than the risk
already presented by Oozie jobs that include Swift credentials. In fact, this
is probaby safer since a user must have direct access to the job directory to
read the credentials written by Sahara.

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

The DIB image for Spark will need to change to include the Hadoop Swift JAR.
There is an existing element for this, this is trivial.

Additionally, we still need a solution to a compatibility issue with the
`jackson-mapper-asl.jar.`

This can be patched by including an additional JAR on the cluster nodes, so we
can conceivably bundle the extra jar with the Spark image as a patch.
Alternatively, the issue might be resolved by upgrading the CDH version (or
Spark assembly version) used in the image.

Sahara-dashboard / Horizon impact
---------------------------------

The Spark job submission form should have an input (checkbox?) which allows a
user to set `edp.spark.adapt_for_swift` to True. Default should be False.

We don’t want to assume that just because Swift paths are passed as arguments
to a Spark job that Sahara should run the wrapper. The job itself may handle
the Swift paths in its own way.

Implementation
==============

Assignee(s)
-----------

Primary assignee: tmckay

Other contributors: croberts

Work Items
----------

* Add a simple SparkWrapper class.
  This is different than the wrapper class developed for Oozie Java actions.

* Update Spark image to include the Hadoop Openstack JAR
* Find a solution to the jackson issue
* Update the UI

* Implement handling of the `edp.spark.adapt_for_swift` option in Sahara.
  This includes generation and upload of the extra XML file, upload of the
  additional utility jar, and alteration of the command generated to invoke
  spark-submit

* Updated node configuration
  All nodes in the cluster should include the Hadoop `core-site.xml` with
  general Swift filesystem configuration. Additionally, spark-env.sh should
  point to the Hadoop `core-site.xml` so that Spark picks up the Swift configs
  and `spark-defaults.conf` needs to set up the executor classpath. These
  changes will allow a user to run Spark jobs with Swift paths manually using
  `spark-submit` from any node in the cluster should they so choose.

Dependencies
============

https://blueprints.launchpad.net/sahara/+spec/edp-improve-compatibility

Testing
=======

Unit tests and integration tests for Spark jobs will identify any regressions
introduced by this change.

Once we have Spark images with all necessary elements included, we can
add an integration test for Spark with Swift URLs.


Documentation Impact
====================

Any user docs describing job submission should be updated to cover the new
option for Spark jobs.

References
==========
