..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===============================================
Add a common HBase lib in hdfs on cluster start
===============================================

https://blueprints.launchpad.net/sahara/+spec/edp-add-hbase-lib

HBase applications written in Java require JARs on the HBase classpath.
Java actions launched from Oozie may reference JARs stored in hdfs
using the `oozie.libpath` configuration value, but there is no
standard HBase directory location in hdfs that is installed with Oozie.

Users can build their own HBase directory in hdfs manually from a cluster
node but it would be convenient if Sahara provided an option to build
the directory at cluster launch time.


Problem description
===================

HBase applications written in Java require the Hbase classpath. Typically,
a Java program will be run this way using /usr/bin/hbase to get the classpath::

  java -cp `hbase classpath`:MyHbaseApp.jar MyHBaseApp

Java jobs launched from EDP are Oozie actions, and there is no way to set
an extra classpath value. Instead, the Oozie solution to this problem is
to create a directory of JAR files in hdfs and then set the
`oozie.libpath` configuration property on the job to that location.
This causes Oozie to make all of the jars in the directory available to the
job.

Sahara currently supports setting the `oozie.libpath` configuration on a job
but there is no existing collection of HBase JARs to reference. Users can log
into a cluster node and build an HBase directory in hdfs manually using bash or
Python.  The steps are relatively simple:

* Run the ``hbase classpath`` command to retrieve the classpath as a string
* Separate the string on the **:** character
* Prune away all paths that do not end in **.jar**
* Upload all of the remaining paths to the designated directory in hdfs

However, it would be relatively simple for Sahara to do this optionally
at cluster creation time for clusters that include HBase services.

Note that the same idea of a shared hdfs directory is used in two
different but related ways in Oozie:

* the `Oozie sharelib` is a pre-packaged collection of JARs released and
  supported as part of Oozie and referenced from a job by
  setting the `oozie.use.system.libpath` configuration parameter to `True`.
  Sahara already sets this option for all Ooozie-based jobs.
  The official Oozie sharelib changes over time, and Oozie uses a timestamp
  naming convention to support upgrades, multiple versions, etc.

* the ability to create an hdfs directory containing JAR files and reference
  it from a job with the `oozie.libpath` configuration parameter is open to
  anyone. This is what is being proposed here. This change in no way touches
  the official `Oozie sharelib`. If Oozie ever adds HBase JARs to the
  system sharelib, we probably will no longer need this feature.

Proposed change
===============

Create a class that can be shared by any provisioning plugins that support
installing the HBase service on a cluster. This class should provide a
method that runs remote commands on a cluster node to:

* Run the ``hbase classpath`` command to retrieve the classpath as a string
* Separate the string on the **:** character
* Prune away all paths that do not end in **.jar**
* Upload all of the remaining paths to pre-determined directory in hdfs

A code sample in the reference section below shows one method of doing this
in a Python script, which could be uploaded to the node and executed via
remote utilities.

The HBase hdfs directory can be fixed, it does not need to be configurable. For
example, it can be "/user/sahara-hbase-lib" or something similar. It should be
readable by the user that runs Hadoop jobs on the cluster. The EDP engine can
query this class for the location of the directory at runtime.

An option can be added to cluster configs that controls the creation of this
hdfs library. The default for this option should be True. If the config option
is True, and the cluster is provisioned with an HBase service, then the hdfs
HBase library should be created after the hdfs service is up and running and
before the cluster is moved to "Active".

A job needs to set the `oozie.libpath` value to reference the library.
Setting it directly presents a few problems:

* the hdfs location needs to be known to the end user
* it exposes more "Oozie-ness" to the end user. A lot of "Oozie-ness" has
  leaked into Sahara's interfaces already but there is no reason to
  add to it.

Instead, we should use an `edp.use_hbase_lib` boolean configuration
parameter to specify whether a job should use the HBase hdfs library. If this
configuration parameter is True, EDP can retrieve the hdfs location
from the utility class described above and set `oozie.libpath` accordingly.
Note that if for some reason an advanced user has already set `oozie.libpath`
to a value, the location of the HBase lib should be added to the value (which
may be a comma-separated list).


Alternatives
------------

* Do nothing.  Let users make their own shared libraries.

* Support Oozie Shell actions in Sahara.

  Shell actions are a much more general feature under consideration
  for Sahara. Assuming they are supported, a Shell action provides
  a way to launch a Java application from a script and the classpath
  can be set directly without the need for an HBase hdfs library.

  Using a Shell action would allow a user to run a script. In a script
  a user would have complete control over how to launch a Java application
  and could set the classpath appropriately.

  The user experience would be a little different.  Instead of just writing
  a Java HBase application and launching it with `edp.use_hbase_lib` set to
  True, a user would have to write a wrapper script and launch that as a
  Shell action instead.

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

We may want a simple checkbox option on the UI for Java actions
to set the `edp.use_hbase_libpath` config so that users don't need to
add it by hand.


Implementation
==============

Assignee(s)
-----------

Primary assignee:
    huichun

Other contributors:
    tmckay


Work Items
----------


Dependencies
============


Testing
=======

An EDP integration test on a cluster with HBase installed would be great test
coverage for this since it involves cluster configuration.

Unit tests can verify that the oozie.libpath is set correctly.

Documentation Impact
====================

We need to document how to enable creation of the shared lib at cluster
creation time, and how to configure a job to reference it


References
==========

Here is a good blog entry on Oozie shared libraries in general.

  http://blog.cloudera.com/blog/2014/05/how-to-use-the-sharelib-in-apache-oozie-cdh-5/

Here is a simple script that can be used to create the lib::

  #!/usr/bin/python
  import sys
  import os
  import subprocess

  def main():
      subprocess.Popen("hadoop fs -mkdir %s" % sys.argv[1], shell=True).wait()
      cp, stderr = subprocess.Popen("hbase classpath",
                                    shell=True,
                                    stdout=subprocess.PIPE,
                                    stderr=subprocess.PIPE).communicate()
      paths = cp.split(':')
      for p in paths:
          if p.endswith(".jar"):
              print(p)
              subprocess.Popen("hadoop fs -put %s %s" % (os.path.realpath(p),
                                                         sys.argv[1]),
                                                         shell=True).wait()

  if __name__ == "__main__":
      main()
