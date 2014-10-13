..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================================
Move the EDP examples from the sahara-extra repo to sahara
==========================================================


https://blueprints.launchpad.net/sahara/+spec/edp-move-examples

Moving the Sahara EDP examples from the sahara-extra repo to
the sahara repo accomplishes several things:

* It eliminates code duplication since the examples are actually
  used in integration tests
* It removes an element from the sahara-extra repo, thereby moving
  us closer to retiring that repo and simplifying our repo structure
* It puts examples where developers are more likely to find it, and
  makes it simpler to potentially bundle the examples with a Sahara
  distribution


Problem description
===================

The goal is to create one unified set of EDP jobs that can
be used to educate users and developers on how to create/run
jobs and can also be used as jobs submitted during integration
testing.


Proposed change
===============

Under the sahara root directory, we should create a new directory::

  sahara/edp-examples

The directory structure should follow a standard pattern (names
are not important per se, this is just an illustration)::

  subdirectory_for_each_example/
    README.rst (what it is, how to compile, etc)
    script_and_jar_files
    src_for_jars/
    how_to_run_from_node_command_line/ (optional)
    expected_input_and_output/ (optional)
  hadoop_1_specific_examples/
    subdirectory_for_each_example
  hadoop_2_specific_examples/
    subdirectory_for_each_example

The integration tests should be modified to pull job files from
the sahara/edp-examples directory.

Here are some notes on equivalence for the current script and jar
files in ``sahara-extra/edp-examples`` against
``sahara/tests/integration/tests/resources``::

  pig-job/example.pig == resources/edp-job.pig
  pig-job/udf.jar == resources/edp-lib.jar
  wordcount/edp-java.jar == resources/edp-java/edp-java.jar

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

Examples won't be found in the sahara-extra repo any longer.
We should perhaps put a README file there that says "We have
moved" for a release cycle.

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

None as yet

Work Items
----------

The problem has several components:

* Move the examples to the sahara repository
* Merge any jobs used by the integration tests into the new
  examples directory to create one comprehensive set
* Provide source code and compilation instructions for any
  examples that currently lack them
* Make the integration tests reference the new directory structure
* Delineate which, if any, examples work only with specific
  Hadoop versions.  Most examples work on both Hadoop 1 and Hadoop 2
  but some do not.  Version-specific examples should be in a subdirectory
  named for the version


Dependencies
============

None


Testing
=======

Testing will be inherent in the integration tests. The change will be
deemed successful if the integration tests run successfully after the
merging of the EDP examples and the integration test jobs.

Documentation Impact
====================

If our current docs reference the EDP examples, those references should
change to the new location.  If our current docs do not reference the
EDP examples, a reference should be added in the developer and/or user
guide.


References
==========

None
