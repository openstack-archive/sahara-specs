..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==============================================
Use Templates for Scenario Tests Configuration
==============================================

https://blueprints.launchpad.net/sahara/+spec/scenario-test-config-template


Users that want to run the scenario tests available in the Sahara repository
need to modify the provided YAML files, which are really template files even
if not marked as such. The usability of this process could be improved by
extending the test runner to read the templates and perform the substitution
of the required variables instead.

Problem description
===================

The scenario tests provided in the Sahara repository are template files
even if they are simply marked as YAML files; this is not clear from the
README.rst file.
Users that want to run them need to manually replace the required variables
(the variables are needed because they depend on the environment: host ip,
credentials, network type, flavors used, etc).
This step needs to be done both by developers and testers, and by wrapper
scripts used to run the script on a CI. The repository of the main Sahara CI,
sahara-ci-config, contains code which replaces the variables:

https://github.com/stackforge/sahara-ci-config/blob/master/slave-scripts/functions-common.sh#L148

Proposed change
===============

The current template files (mostly under etc/sahara-ci right now) need to be
properly identified as templates. The chosen format is Mako, because it is
already a dependency for the scenario test runner (runner.py).
The files will be marked using a special suffix (file.yaml.mako)
and the variables used will be converted in Mako format.
The usage of templating would be limited to variable replacement, which means
no logic in the templates.

The test runner will continue to handle normal YAML files as usual, in
addition to template files.

runner.py will also take a simply INI-style file with the values for
the variables used in the template. It will be used by the runner
to generate the real YAML files used for the input (in addition to
the normal YAML files, if specified).

A missing value for some variable (key not available in the INI file)
will raise an exception and lead to the runner termination.
This differs from the current behavior of sahara-ci-config code, where a
missing values just prints a warning, but given that this would likely bring
to a failure, enforcing an early termination limit the resource consumption.

The current sahara-ci-config code allows to specify more details for the
replacement variables, like specific end keys where to stop the match for
a certain key, but they are likely not needed with a proper choice of names
for the variables.

Finally, sahara/tests/scenario/README.rst should be changed to document the
currently used variables and the instruction on how to feed the key/value
configuration file.
The code in sahara-ci-config should be changed as well to create such
configuration file and to use the new names of the template files for the
respective tests.

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

CI systems which runs tox -e scenario should be updated to use the new
filenames.

Developer impact
----------------

Developers/QE running tests need to use the new template names.

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
  ltoscano

Work Items
----------

The change will be implemented as follow:

1. allow runner.py to use mako template files as input;
2. copy the existing files to the new name and use non-ambiguous variable
   names for templates;
3. change sahara-ci scripts to use to set the new variables and use the
   renamed template files;
4. remove the old yaml files;
5. (optional) clean up unnedded code (insert_scenario_value, etc)

Repositories affected:
- sahara: 1, 2, 4
- sahara-ci-config: 3, 5


Dependencies
============

None

Testing
=======

Successful CI run will ensure that the new code did not regress the existing
scenarios.


Documentation Impact
====================

sahara/tests/scenario/README.rst needs to be updated.


References
==========

None

