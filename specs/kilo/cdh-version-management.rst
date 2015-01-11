..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

============================================
Better Version Management in Cloudera Plugin
============================================

https://blueprints.launchpad.net/sahara/+spec/cdh-version-management

This specification proposes to manage different versions of CDH in Cloudera
plugin in a better solution.

Problem description
===================

Current situation of CDH version management in CDH plugin:

* We do not have version control for CDH plugin at all.
* Due to backward support requirement, all existing codes need to support
  CDH version from 5.0.0 to latest (currently it is 5.3.0).

This will cause below issues:

* When new packages are released, we do not know whether our plugin still
  can work. In fact nobody can ensure that.
* Our work have to be based on a non-static environment.
* Backward-compatibility is desired, but it is not ensured at all. For
  example, we are not sure tomorrow whether a new released CDH package will
  be compatible with old plugin codes or configuration files.
* CI-test results are not stable, so we cannot always turn it into voting.
* If new released package version bring issues, it can block all developers
  even if they do not work on this version.
* When we do not want to support some obsolete versions, we cannot easily
  remove it.

Proposed change
===============

* Add package version constraints, and give a key packages version list to
  support CDH5.3.0 (or 5.2.0), and base our development only on this.
* Freeze the package to prevent later CDH release come in.
* Add sub-directories in plugin/cdh, like 5_3_0 for CDH5.3.0 to support
  different versions of CDH. As we did for vanilla and hdp plugins.
* We also need to do some modifications in sahara-image-elements project to
  support different versions of CDH image build.
* We need to change sahara-ci-config project to break current cdh tests into
  several tests for different CDH versions. All new added versions should not
  break old version tests. For example, current
  gate-sahara-nova-direct-cdh_ubuntu-aio will be separated into
  sahara-nova-direct-cdh_5.0_ubuntu-aio and
  sahara-nova-direct-cdh_5.3.0_ubuntu-aio.
* Break sahara/tests/integration/tests/gating/test_cdh_gating.py into
  several files, like test_cdh5_0_0_gating.py and test_cdh5_3_0_gating.py to
  support different CDH versions.
* We need not ensure backward compatibility. E.g., CDH5.3.0 codes are not
  required to work for CDH5.0.0. We may add some features and codes only for
  later version of CDH, which was not supported by former CDH version.
* If we want to add more CDH versions in the future, we need to open a BP to
  do this. For example, if we want to support CDH 5.4.0 in the future, below
  works are required:

    * Add a directory 5_3_0 including codes to support CDH 5.3.0 in
      plugin/cdh.
    * Modify scripts in sahara/image-elements/elements/hadoop_cloudera, to
      add functions to install packages for CDH 5.4.0, while still keeping
      functions installing packages for former CDH versions.
    * Add a test_cdh5_4_0_gating.py in sahara/tests/integration/tests/gating
      directory for integration test.
    * Add a new ci-test item in sahara-ci-config, like
      gate-sahara-nova-direct-cdh_5.4.0_ubuntu-aio. We can set this item as
      non-voting at the beginning, and after it is tuned and verified ok, we
      set it back to voting.

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
  ken chen

Other contributors:
  ken chen

Work Items
----------

The work items will include:

* Change directory structure in sahara/sahara/plugins/cdh to add 5_3_0 and
  5_0_0 for CDH5.0.0 and 5.3.0 (We will support those two versions as the
  first step).
* Retain common codes for each version in the original place, and move
  version specific codes and files into their own directory.
* Add codes in sahara-image-elements/elements/hadoop-cloudera to support
  installing different package groups for different CDH versions.
* Add different test items in sahara/tests/integration/tests/gating directory
  to support different CDH versions
* Add item in sahara-ci-configs project for different CDH versions. At first
  we mark it as non-voting. After it is verified, we can mark it as voting.
* Test and evaluate the change.

Dependencies
============

None

Testing
=======

Create clusters of different CDH versions one by one, and do integration tests
for each of them.

Documentation Impact
====================

None

References
==========

None
