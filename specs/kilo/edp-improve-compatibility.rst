==========================================
[EDP] Improve Java type compatibility
==========================================

https://blueprints.launchpad.net/sahara/+spec/edp-improve-compatibility

Currently, EDP MapReduce (Java type) examples must add modifications
to be able to use from a java action in an Oozie workflow.

This bp aims that users can migrate from other Hadoop cluster to Sahara
without any modifications into their applications.


Problem description
===================

Users need to modify their MapReduce programs as below:

* Add `conf.addResource` in order to read configuration values from
  the `<configuration>` tag specified in the Oozie workflow::

    // This will add properties from the <configuration> tag specified
    // in the Oozie workflow.  For java actions, Oozie writes the
    // configuration values to a file pointed to by ooze.action.conf.xml
    conf.addResource(new Path("file:///",
                              System.getProperty("oozie.action.conf.xml")));

* Eliminate `System.exit` for following restrictions of Oozie's Java
  action.
  e.g. `hadoop-examples.jar` bundled with Apache Hadoop has been used
  `System.exit`.

First, users would try to launch jobs using examples and/or
some applications executed on other Hadoop clusters (e.g. Amazon EMR).
We should support the above users.


Proposed change
===============

We will provide a new job type, called Java EDP Action, which overrides
the Main class specified by `main_class`.
The overriding class adds property and calls the original main method.
The class also catches an exception that is caused by `System.exit`.


Alternatives
------------

According to Oozie docs, Oozie 4.0 or later provides the way of overriding
an action's Main class (3.2.7.1).
The proposing implementation is more simple than using the Oozie feature.
(We will implement this without any dependencies of Oozie library.)


Data model impact
-----------------

None

REST API impact
---------------

None

Other end user impact
---------------------

Users will no longer need to modify their applications to use EDP.

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

sahara-dashboard / horizon needs to add this new job type.


Implementation
==============

Assignee(s)
-----------

Primary assignee: Kazuki Oikawa (k.oikw)

Other contributors: Yuji Yamada (yamada-yuji)


Work Items
----------

* Add new job type (`Java.EDP`)

  * `Java.EDP` will be subtype of `Java`

  * Implement of uploading jar file of overriding class to HDFS

  * Implement of creating the `workflow.xml`

* Implement the overriding class


Dependencies
============

None


Testing
=======

We will add a integration test.
This test checks whether WordCount example bundled with Apache Hadoop
executes successfully.


Documentation Impact
====================

If EDP examples use this feature, the docs need to update.


References
==========

* Java action in Oozie http://oozie.apache.org/docs/4.0.0/WorkflowFunctionalSpec.html#a3.2.7_Java_Action
