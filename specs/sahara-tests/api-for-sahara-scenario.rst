===================================================================
Add an API to sahara-scenario for integration to another frameworks
===================================================================

https://blueprints.launchpad.net/sahara-tests/+spec/api-for-scenario-tests

Sahara scenario is the main tool for testing Sahara and we can provide
API for using this framework in another tools/frameworks/tests.

Problem description
===================

When you perform testing of Sahara from another framework, you can't reuse
created scenarios and need to create similar code in another framework with
new scenarios for Sahara. I think, that we can implement simple Python API for
running Sahara scenarios from default templates. It will be useful, for
example in frameworks with destructive tests or something else.

Proposed change
===============

Refactor current `runner.py` module, pull out code
in `sahara_tests/scenario/utils.py` for reusing it in API methods.

I propose to move code with:

- creation of tmp files,
- working with .mako,
- running tests,
- merging/generation of scenario files.

In `runner.py` we should leave only calls of this methods, for more convenient
usage. The next step is adding the directory `api` in `sahara_tests/scenario`
with files for implementing API.

I think, we need to add file `api.py` with methods for external usage:

- def run_scenario(plugin, version, release=None)

and `base.py` with preparing of files for running and auxiliary methods in
the future.

In future we can separate one scenario into steps:

- create node group template;
- create cluster template;
- create cluster;
- create data sources;
- create job binaries;
- perform EDP;

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
  esikachev

Work Items
----------

- move methods from `runner.py`
- add the ability to run tests via API
- add the job for testing API

Dependencies
============

None

Testing
=======

We can create separate job on Sahara-ci with custom script for checking API
calls and correct performing of actions.

Documentation Impact
====================

The API implementation should be mentioned in Sahara-tests docs.

References
==========

None
