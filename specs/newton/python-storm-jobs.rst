..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=============================================
Allow creation of python topologies for Storm
=============================================

In order to allow user to develop storm topologies that we can also can call
storm jobs in a pure python form we are adding support to Pyleus in Sahara.
Pyleus is a framework that allows the creation of Storm topologies in
python and uses yaml to wire how the flow is going to work.


Problem description
===================

https://blueprints.launchpad.net/sahara/+spec/python-storm-jobs

Storm is the plugin in Sahara responsible for real time processing. Storm is
natively written in Java, being the common languange to write topologies.
Storm allows topologies to be written in different languages, including
python, but the default way to implement this still requires a java shell
combining the python components together being not very pythonic.


Proposed change
===============

We are OpenStack and we love our python, so in order to allow OpenStack users
to create a submit python written topologies we propose to integrate the
Pyleus framework into the Storm plugin.
Pyleus allows the user to create Storm topologies components in python and
it provides an abstraction to help that construction, making the
implementation easier. Also it uses a yaml file that will be used to compile
the topology. The final object of the compilation is a jar file containing
the python written topology.
The change we need to do is to integrate pyleus command line into the Storm
plugin to submit the topologies using its CLI instead of Storm's. The
overall UX will remain the same, since the user will upload a jar file and
start a topology. We will create a new job type for Storm called pyleus so
the plugin can handle the new way of submitting the job to the cluster.

The command line will like this:

* pyleus submit [-n NIMBUS_HOST] /path/to/topology.jar
* pyleus kill [-n NIMBUS_HOST] TOPOLOGY_NAME
* pyleus list [-n NIMBUS_HOST]


Alternatives
------------

The option is to leave Storm as it is, accepting only default jobs java or
python.

Data model impact
-----------------

None.

REST API impact
---------------

There will be a minor REST API impact since we are introducing a new Job Type.

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

We will need to install pyleus on the Storm images.

Sahara-dashboard / Horizon impact
---------------------------------

Minor changes will be made to add new Job Type.


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  tellesmvn

Other contributors:
  None

Work Items
----------

1) Create new Job type.
2) Change Storm plugin the deal with the new job type.
3) Implement tests for this feature.

Dependencies
============

None.


Testing
=======

To this point only unit tests will be implemented.


Documentation Impact
====================

This feature should be documented in the user doc.

References
==========

None.