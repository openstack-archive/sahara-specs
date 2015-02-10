..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
New style logging
==========================================

https://blueprints.launchpad.net/sahara/+spec/new-style-logging

Rewrite Sahara logging in unified OpenStack style proposed by
https://blueprints.launchpad.net/nova/+spec/log-guidelines

Problem description
===================

Now log levels and messages in Sahara are mixed and don't match the OpenStack
logging guideliness.

Proposed change
===============

The good way to unify our log system would be to follow the major guideliness.
Here is a brief description of log levels:

* Debug: Shows everything and is likely not suitable for normal production
  operation due to the sheer size of logs generated (e.g. scripts executions,
  process execution, etc.).
* Info: Usually indicates successful service start/stop, versions and such
  non-error related data. This should include largely positive units of work
  that are accomplished (e.g. service setup, cluster start, successful job
  execution).
* Warning: Indicates that there might be a systemic issue;
  potential predictive failure notice (e.g. job execution failed).
* Error: An error has occurred and an administrator should research the event
  (e.g. cluster failed to start, plugin violations of operation).
* Critical: An error has occurred and the system might be unstable, anything
  that eliminates part of Sahara's intended functionality; immediately get
  administrator assistance (e.g. failed to access keystone/database, plugin
  load failed).

Here are examples of LOG levels depending on cluster execution:

* Script execution:

.. code:: python

    LOG.debug("running configure.sh script")


..

* Cluster startup:

.. code:: python

    LOG.info(_LI("Hadoop stack installed successfully."))


..

* Job failed to execute:

.. code:: python

    LOG.warning(_LW("Can't run job execution {job} (reason: {reason}")).format(
                job = job_execution_id, reason = ex)


..

* HDFS can't be configured:

.. code:: python

    LOG.error(_LE('Configuring HDFS HA failed. {reason}')).format(
              reason = result.text)


..

* Cluster failed to start:

.. code:: python

    LOG.error(_LE('Install command failed. {reason}')).format(
              reason = result.text)


..

Additional step for our logging system should be usage of pep3101 as unified
format for all our logging messages. As soon as we try to make our code more
readable please use {<smthg>} instead of {0} in log messages.

Alternatives
------------

We need to follow OpenStack guideliness, but if needed we can move plugin logs
to DEBUG level instead of INFO. It should be discussed separately in each case.

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

None

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  starodubcevna

Work Items
----------

* Unify existing logging system
* Unify logging messages
* Add additional logs if needed

Dependencies
============

None

Testing
=======

None

Documentation Impact
====================

None

References
==========

https://blueprints.launchpad.net/nova/+spec/log-guidelines
https://www.python.org/dev/peps/pep-3101/
