..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===============================
Feature sets for scenario tests
===============================

The runner and the templates for scenario tests do not provide an easy
way to specify the dependency between a certain item (i.e. an EDP job for
a certain cluster) and its configuration (i.e. credentials, the definition
of the job). This proposal tries to address the problem.

Problem description
===================

A key feature of sahara-scenario is the support for test templates
They support parameters which allow tester to describe different
test scenarios without writing multiple copies of test templates
which differs only by few arguments.
This flexibility is somehow limited when testing features like S3 support.
Testing S3 integration requires few additional details
to be specified when configuring the tests:

* the credentials required to access the S3 API;

* the definitions of the new EDP jobs which uses S3;

* for each cluster where the jobs needs to be executed,
  the name of the EDP job.

The first two items can be easily added to new files
that can be specified as argument for the sahara-scenario
command (for example `credentials_s3.yaml.mako` and
`edp_s3.yaml.mako`). The keys that they contain
(`credentials` and `edp_jobs_flow`) are merged together
by the runner.
Their content may also be added directly to the existing
YAML files (`credentials.yaml.mako` and `edp_s3.yaml.mako`)
but that would mean adding a long list of default values
for all the arguments in the template.

The third item is more complicated to model, because
there is no easy way to override an item in a `cluster`
element, which is a list, not a dictionary.

While it could be possible to introduce a command line parameter
to override the items in a specific cluster, that would
still leave up to the user to remember to specify all
the required bits to enable S3 testing.

A more general solution to the problem is the definition
of feature sets for testing.


Proposed change
===============
The user of sahara-scenario would only need to pass an argument like:

::

    sahara-scenario ... --feature s3 ...

If the `-p` and `-v` arguments are specified, for each `feature` argument
`sahara-scenario` will include the files `credentials_<feature>.yaml.mako`
and `edp_<feature>.yaml.mako`, if they exist.

In addition, from the list of EDP jobs specified for all enabled clusters,
all the items marked with `feature: s3` will be selected.

This means that items without the `feature` tag will always be executed,
while items with `feature` will be executed only when the associated
feature(s) are selected.

The initial implementation will focus on EDP jobs, but other items
may benefit from the tagging.

Alternatives
------------

Use conditional statements in the YAML file. But the author
of this spec requested multiple times to keep the YAML files
free of business logic and purely declarative instead
when the `sahara-scenario` initial spec was discussed.

Another possible solution is the duplication of the YAML templates,
which is going against maintainability and easiness of usage.

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

The new test runner will be backward compatible with the old templates,
but new templates will require the new runner, but this should not be
a problem.

Developer impact
----------------

None.

Image Packing impact
--------------------

None.

Sahara-dashboard / Horizon impact
---------------------------------

None.


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  ltoscano

Work Items
----------

* extend the runner to accept the `feature` argument

* add new EDP jobs and add the feature marker to the EDP jobs
  which need it, extending the existing attempt
  of S3 testing (see :ref:`spec-references`.)
  which is an early attempt to solve the issue.

Dependencies
============

None.


Testing
=======

The new argument and the merging of values will be covered by unit tests.
The regression testing of will be covered by the `sahara-tests-scenario`
jobs in the gate.


Documentation Impact
====================

The new arguments and its usage will be documented from the user and
the test writer point of view.

.. _spec-references:

References
==========

Initial work to support S3 testing:

* https://review.openstack.org/610920

* https://review.openstack.org/590055
