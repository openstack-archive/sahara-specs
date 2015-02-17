..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=====================================
[EDP] Add Oozie Shell Action job type
=====================================

https://blueprints.launchpad.net/sahara/+spec/add-edp-shell-action

Oozie shell actions provide a great deal of flexibility and will
empower users to easily customize and extend the features of
Sahara EDP as needed. For example, a shell action could be used
to manage hdfs on the cluster, do pre or post processing for
another job launched from Sahara, or run a data processing job
from a specialized launcher that does extra configuration not
otherwise available from Sahara (ie, setting a special classpath
for a Java job).


Problem description
===================

Today if a user finds a limitation in Sahara EDP that prevents them
from performing desired processing they have a few choices:

* Log into the cluster nodes with ssh using the specified keypair
  and perform processing by hand. This may not be available to all
  users.

* Write a Java application to do the custom processing and run it
  as a Java job in EDP. This can work, however we as developers know
  that Java is not often used as a scripting language; it's a little
  too heavyweight and not everyone knows it.

* Modify the Sahara source. A savvy user might extend Sahara EDP
  themselves to get the desired functionality. Howerver, not everyone
  is a developer or has the time to understand Sahara enough to do this.

* Submit a bug or a blueprint and wait for the Sahara team to address it.

However, the existence of shell actions would empower users to easily
solve their own problems. With a shell action, a user can bundle files
with a script written in bash, Python, etc and execute it on the cluster.

Here is a real-world example of a case that could be easily solved
with a shell action:

https://blueprints.launchpad.net/sahara/+spec/edp-add-hbase-lib

In the above blueprint we are calling for additional features in Sahara
as a convenience for users, but with a shell action a user could solve this
problem on their own. A simple bash script can be written to launch a Java
application like this:

.. code::

   #!/bin/bash
   java -cp HBaseTest.jar:`hbase classpath` HBaseTest

In this case a user would associate the script and the Java application
with the shell job as job binaries and Sahara would execute the script.

In a similar case, Sahara EDP itself uses a Python wrapper around
spark-submit to run Spark jobs. A shell action makes these kinds of
launchers easily available to end users.


Proposed change
===============

Add a `Shell` job type that is implemented by the Oozie EDP engine.
(It is possible that other EDP engines, such as the Spark or Storm
engines, could share a basic shell command implementation for such
jobs but that would be another CR).

Shell jobs can use the existing `mains` and `libs` fields in a job
execution. The script identified in `mains` will be the script that
Sahara runs and the files in `libs` will be supporting files bundled
in the working directory by Oozie.

As with other job types, shell actions will support `configs` and
`args` passed in the existing `job_configs` field of a job execution.
Values in `configs` are specified in the Oozie workflow with
**<configuration>** tags and will be available in a file created by Oozie.
The `args` are specified with the **<argument>** tag and will be passed
to the shell script in order of specification.

In the reference section below there is a simple example of a shell
action workflow. There are three tags in the worflow that for Sahara's
purposes are unique to the `Shell` action and should be handled by
Sahara:

* **<exec>script</exec>**
  This identifies the command that should be executed by the shell action.
  The value specified here will be the name of the script idenfied in `mains`.
  Technically, this can be any command on the path but it is probably
  simpler if we require it to be a script. Based on some experimentation,
  there are subtleties of path evaluation that can be avoided if a script
  is run from the working directory

* **<file>support.jar</file>**
  This identifies a supporting file that will be included in the working
  directory. There will be a <file> tag for every file named in `libs`
  as well as the script identified in `mains`.

  (Note that the <file> tag can be used in Oozie in any workflow, but
  currently Sahara does not implement it at all. It is necessary for the
  shell action, which is why it's mentioned here. Whether or not to add
  general support for <file> tags in Sahara is a different discussion)

* **<env-var>NAME=VALUE</env-var>**
  The env-var tag sets a variable in the shell's environment. Most likely
  we can use the existing `params` dictionary field in `job_configs` to
  pass env-var values even if we want to label them as "environment
  variables" in the UI.


Alternatives
------------

Do nothing.


Data model impact
-----------------

This change adds a new job type, but since job types are stored as strings
it should not have any data model impact.

REST API impact
---------------

Only a change in validation code for job type

Other end user impact
---------------------

None

Deployer impact
---------------

There may be security considerations related to `Shell Action Caveats`,
bullet number 2, in the URL listed in the reference section.

It is unclear whether or not in Sahara EDP the user who started the TaskTracker
is different than the user who submits a workflow. This needs investigation --
how is Sahara authenticating to the Oozie client? What user is the Oozie
server using to deploy jobs?

Developer impact
----------------

None

Sahara-image-elements impact
----------------------------

None

Sahara-dashboard / Horizon impact
---------------------------------

We would need a new form for a Shell job type submission. The form should allow
specification of a main script, supporting libs, configuration values,
arguments, and environment variables (which are 100% analagous to params from
the perspective of the UI)

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  egafford

Other contributors:
  tmckay

Work Items
----------

* Investigate user issue mentioned above (who is the user that runs
  shell actions in Sahara and what are the implications?)
* Add a Shell job type and an implementation in the Oozie EDP engine
  components under the `workflow_creator` directory
* Update job validation routines to handle the Shell job type
* Add an integration test for Shell jobs
* Update the EDP documentation to describe the Shell job type
* Add a UI form for Shell job submission

Dependencies
============

None

Testing
=======

* Unit tests to cover creation of the Shell job
* Integration tests to cover running of a simple shell job


Documentation Impact
====================

The EDP sections of the documentation need updating


References
==========

http://blog.cloudera.com/blog/2013/03/how-to-use-oozie-shell-and-java-actions/

A simple Shell action workflow looks like this::

  <workflow-app xmlns='uri:oozie:workflow:0.3' name='shell-wf'>
    <start to='shell1' />
    <action name='shell1'>
        <shell xmlns="uri:oozie:shell-action:0.1">
            <job-tracker>${jobTracker}</job-tracker>
            <name-node>${nameNode}</name-node>
            <configuration>
                <property>
                  <name>mapred.job.queue.name</name>
                  <value>default</value>
                </property>
            </configuration>
            <exec>doit.sh</exec>
            <argument>now</argument>
            <env-var>VERSION=3</env-var>
            <file>HBaseTest.jar</file>
            <file>doit.sh</file>
        </shell>
        <ok to="end" />
        <error to="fail" />
    </action>
    <kill name="fail">
        <message>oops!</message>
    </kill>
    <end name='end' />
  </workflow-app>
