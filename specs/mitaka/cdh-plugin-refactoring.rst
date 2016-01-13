..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===============================
Code refactoring for CDH plugin
===============================

https://blueprints.launchpad.net/sahara/+spec/cdh-plugin-refactoring

This spec is to do some refactoring to the code to allow easier support to new
versions in the future.

Problem description
===================

CDH plugin contains many duplicated code. Current implementation extracts some
general and base behavior of the plugin, and each version has its own
implementation for something not included in base classes and modules. But
there are many overlaps between versions because of the downward compatibility.
For example, sahara.plugins.cdh.v5.config_helper extends
sahara.plugins.cdh.db_helper, but functions such as get_plugin_configs are
written again in sahara.plugins.cdh.v5_3_0.config_helper.

And currently the low test coverage of CDH plugin makes it hard to guarantee
the quality of the new code after refactoring. So some new unit test cases need
to be added. And some old test cases may be altered according to refactoring.

Proposed change
===============

For duplicate codes in each version of plugins, move them to the base class.
In validation module, function validate_cluster_creating is too long to be read
easily. Seprate it into serveral small clearly functions.
We can encapsulate funtions in module into a class for better extensibility.
ClouderaUtils and deploy modules are not going to be changed util CDH v5 are
totally removed, because these modules' codes in v5 are quite different
from other versions.

Alternatives
------------

There is another way, just let a new version extends an old one instead of
all extends from base. This may bring problems when deperate an old version.

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
  jxwang

Work Items
----------

This will require following changes:

* move the duplicate codes to base class.
  Files need to be modified:
  sahara/plugin/cdh/version/edp_engine.py: EdpOozieEngine, EdpSparkEngine
  sahara/plugin/cdh/version/plugin_utils.py: PluginUtils
  sahara/plugin/cdh/version/versionhandler.py: VersionHandler
  sahara/plugin/cdh/version/config_helper.py
  sahara/plugin/cdh/version/validation.py
* Seperate Validation.validate_cluster_creating function
* Add unit test case for low covered modules.
* Remove useless test cases dedicated for refactoring
  useless test cases in:
  sahara/tests/unit/plugins/cdh/v5/test_versionhandler.py
  sahara/tests/unit/plugins/cdh/v5_3_0/test_versionhandler.py
  sahara/tests/unit/plugins/cdh/v5_4_0/test_versionhandler.py


Dependencies
============

None

Testing
=======

Before starting refactoring, keep current scenario test and provide new unit
tests to ensure the CDH works as well as before. For each version, unit tests
are aslo added individually. Test cases are written for the codes before
refactoring, so we may need a little changes for new codes.

Documentation Impact
====================

None

References
==========

None
