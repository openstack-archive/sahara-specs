..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=========
Storm EDP
=========

https://blueprints.launchpad.net/sahara/+spec/storm-edp

This blueprint aims to implement EDP for Storm. This will require a Storm Job
Type.

Problem description
===================

Sahara needs an EDP implementation to allow the submission of Storm Jobs.


Proposed change
===============

The implementation of the EDP engine will have 3 basic functions:

* run_job()
* get_job_status()
* cancel_job(kill=False)

This methods are mapped to Storm's:
* deploy_toplogy (i.e. storm jar topology-jar-path class ...)
* storm list (i.e. storm list)
* storm deactivate (i.e storm deactivate topology-name)
* storm kill (i.e. storm kill topology-name)

The second part of this implementation is to adapt the UI to allow Storm Job
submission.

Alternatives
------------

We may be able to submit Storm jobs as Java Job but it is better for the user
to have a specific Storm Job.

Data model impact
-----------------

None.

REST API impact
---------------

None.

Other end user impact
---------------------

None.

Deployer impact
---------------

None.

Developer impact
----------------

None.

Sahara-image-elements impact
----------------------------

None.

Sahara-dashboard / Horizon impact
---------------------------------

Sahara needs to adapt its UI to allow creation of Storm Jobs. A draft was done
by crobertsrh and can be found in https://review.openstack.org/#/c/112408/4

The main changes in the UI will be:
* Box for the user to define the main class to be executed
* Box for the user to give parameters (if applicable)
* Buttons to control job execution (Start, Stop, Kill, View Status)
* Since it is possible to have more than one job executing in the same topology
the control can be done by job or by topology. In the second case the user
will have to choose between the jobs in the topology to control.


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  tellesmvn

Other contributors:
  tmckay (primary review)
  crobertsrh

Work Items
----------

* Implement Storm Job Type
* Implement EDP engine for Storm
* Implement Unit tests
* Implement integration tests

Dependencies
============

None.


Testing
=======

First we will implement Unit Tests that follow the example from Spark found in
https://github.com/openstack/sahara/blob/master/sahara/tests/unit/service/edp/spark/test_spark.py
And also implement the integration tests

Documentation Impact
====================

The documentation needs to be updated with information about Storm EDP and also
about Storm Job Type.


References
==========

* `Etherpad <https://etherpad.openstack.org/p/juno-summit-sahara-edp>`
* `Storm Documentation <http://storm.incubator.apache.org/>`
