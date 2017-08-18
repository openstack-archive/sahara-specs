..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==================================
[EDP] Add a new job-types endpoint
==================================

https://blueprints.launchpad.net/sahara/+spec/edp-job-types-endpoint

Add a new job-types endpoint that can report all supported job types for
a Sahara instance and which plugins support them.

Problem description
===================

There are currently two problems around job types in Sahara that impact
user experience:

* The current */jobs/config-hints/<job_type>* endpoint is not adequate for
  providing configuration hints because it does not take into account plugin
  type or the framework version. This endpoint was meant to give users a list
  of likely configuration values for a job that they may want to modify
  (at one point the UI incorporated the hints as well). Although some config is
  common across multiple plugins, hints need to be specific to the plugin and
  the framework version in order to be useful. The endpoint does not take
  a cluster or plugin argument and so hints must be general.

* A user currently has no indicator of the job types that are actually
  available from a Sahara instance (the UI lists them all). The set of
  valid job types is based on the plugins loaded for the current instance.
  Furthermore, not all job types will be available to run on all
  clusters launched by the user because they are plugin dependent.

These problems should be solved without breaking backward compatibility in
the REST API.

Proposed change
===============

Add a new endpoint that will indicate for the running Sahara instance
which job types are supported by which versions of which plugins.
Optionally, plugin-and-version-specific config hints will be included
for each supported job type.

Because config hints can be very long, they will not be included in a
response by default.  A query string parameter will be used to indicate
that they should be included.

The endpoint will support the following optional query strings for filtering.
Each may be used more than once to query over a list of values, for example
`type=Pig&type=Java`:

* **type**
  A job type to consider. Default is all job types.

* **plugin**
  A plugin to consider.  Default is all plugins.

* **version**
  A plugin version to consider. Default is all versions.

The REST API method is specified in detail below under *REST API impact*.

We will need two new optional methods in the `Plugin SPI`. This information
ultimately comes from the EDP engine(s) used by a plugin but we do
not want to actually allocate an EDP engine object for this so the
existing **get_edp_engine()** will not suffice (and besides, it requires
a cluster object)::

  @abc.abstractmethod
  def get_edp_job_types(self, versions=[]):
      return []

  @abc.abstractmethod
  def get_edp_config_hints(self, job_type, version):
      return {}

These specific methods are mentioned here because they represent a
change to the public `Plugin SPI`.

Alternatives
------------

Fix the existing */jobs/config-hints* endpoint to take a cluster id or a
plugin-version pair and return appropriate config hints. However, this
would break backward compatibility.

Still add an additional endpoint to retrieve the supported job types
for the Sahara instance separate from config hints.

However, it makes more sense to deprecate the current config-hints interface
and add the new endpoint which serves both purposes.

Data model impact
-----------------

None

REST API impact
---------------

Backward compatibility will be maintained since this is a new endpoint.

**GET /v1.1/{tenant_id}/job-types**

Normal Response Code: 200 (OK)

Errors: none

Indicate which job types are supported by which versions
of which plugins in the current instance.

**Example**
    **request**

    .. sourcecode:: text

        GET http://sahara/v1.1/775181/job-types

    **response**

    .. sourcecode:: http

        HTTP/1.1 200 OK
        Content-Type: application/json

    .. sourcecode:: json

        {
            "job_types": [
                {
                    "name": "Hive",
                    "plugins": [
                        {
                            "description": "The Apache Vanilla plugin.",
                            "name": "vanilla",
                            "title": "Vanilla Apache Hadoop",
                            "versions": {
                                "1.2.1": {}
                            }
                        },
                        {
                            "description": "The Hortonworks Sahara plugin.",
                            "name": "hdp",
                            "title": "Hortonworks Data Platform",
                            "versions": {
                                "1.3.2": {},
                                "2.0.6": {}
                            }
                        }
                    ]
                },
                {
                    "name": "Java",
                    "plugins": [
                        {
                            "description": "The Apache Vanilla plugin.",
                            "name": "vanilla",
                            "title": "Vanilla Apache Hadoop",
                            "versions": {
                                "1.2.1": {}
                            }
                        },
                        {
                            "description": "The Hortonworks Sahara plugin.",
                            "name": "hdp",
                            "title": "Hortonworks Data Platform",
                            "versions": {
                                "1.3.2": {},
                                "2.0.6": {}
                            }
                        }
                    ]
                },
                {
                    "name": "MapReduce",
                    "plugins": [
                        {
                            "description": "The Apache Vanilla plugin.",
                            "name": "vanilla",
                            "title": "Vanilla Apache Hadoop",
                            "versions": {
                                "1.2.1": {}
                            }
                        },
                        {
                            "description": "The Hortonworks Sahara plugin.",
                            "name": "hdp",
                            "title": "Hortonworks Data Platform",
                            "versions": {
                                "1.3.2": {},
                                "2.0.6": {}
                            }
                        }
                    ]
                },
                {
                    "name": "MapReduce.Streaming",
                    "plugins": [
                        {
                            "description": "The Apache Vanilla plugin.",
                            "name": "vanilla",
                            "title": "Vanilla Apache Hadoop",
                            "versions": {
                                "1.2.1": {}
                            }
                        },
                        {
                            "description": "The Hortonworks Sahara plugin.",
                            "name": "hdp",
                            "title": "Hortonworks Data Platform",
                            "versions": {
                                "1.3.2": {},
                                "2.0.6": {}
                            }
                        }
                    ]
                },
                {
                    "name": "Pig",
                    "plugins": [
                        {
                            "description": "The Apache Vanilla plugin.",
                            "name": "vanilla",
                            "title": "Vanilla Apache Hadoop",
                            "versions": {
                                "1.2.1": {}
                            }
                        },
                        {
                            "description": "The Hortonworks Sahara plugin.",
                            "name": "hdp",
                            "title": "Hortonworks Data Platform",
                            "versions": {
                                "1.3.2": {},
                                "2.0.6": {}
                            }
                        }
                    ]
                }
            ]
        }

The job-types endpoint returns a list. Each item in the list is
a dictionary describing a job type that is supported by the
running Sahara. Notice for example that the *Spark* job type is missing.

Each job type dictionary contains the name of the job type and
a list of plugins that support it.

For each plugin, we include the basic identifying information and then
a `versions` dictionary. Each entry in the versions dictionary has
the name of the version as the key and the corresponding config hints
as the value. Since this example did not request config hints, the
dictionaries are empty.

Here is an example of a request that uses the plugin and version filters:

**Example**
    **request**

    .. sourcecode:: text

        GET http://sahara/v1.1/775181/job-types?plugin=hdp&version=2.0.6

    **response**

    .. sourcecode:: http

        HTTP/1.1 200 OK
        Content-Type: application/json

    .. sourcecode:: json

        {
            "job_types": [
                {
                    "name": "Hive",
                    "plugins": [
                        {
                            "description": "The Hortonworks Sahara plugin.",
                            "name": "hdp",
                            "title": "Hortonworks Data Platform",
                            "versions": {
                                "2.0.6": {}
                            }
                        }
                    ]
                },
                {
                    "name": "Java",
                    "plugins": [
                        {
                            "description": "The Hortonworks Sahara plugin.",
                            "name": "hdp",
                            "title": "Hortonworks Data Platform",
                            "versions": {
                                "2.0.6": {}
                            }
                        }
                    ]
                },
                {
                    "name": "MapReduce",
                    "plugins": [
                        {
                            "description": "The Hortonworks Sahara plugin.",
                            "name": "hdp",
                            "title": "Hortonworks Data Platform",
                            "versions": {
                                "2.0.6": {}
                            }
                        }
                    ]
                },
                {
                    "name": "MapReduce.Streaming",
                    "plugins": [
                        {
                            "description": "The Hortonworks Sahara plugin.",
                            "name": "hdp",
                            "title": "Hortonworks Data Platform",
                            "versions": {
                                "2.0.6": {}
                            }
                        }
                    ]
                },
                {
                    "name": "Pig",
                    "plugins": [
                        {
                            "description": "The Hortonworks Sahara plugin.",
                            "name": "hdp",
                            "title": "Hortonworks Data Platform",
                            "versions": {
                                "2.0.6": {}
                            }
                        }
                    ]
                }
            ]
        }


Here is another example that enables config hints and also filters by plugin,
version, and job type.

**Example**
    **request**

    .. sourcecode:: text

        GET http://sahara/v1.1/775181/job-types?hints=true&plugin=hdp&version=1.3.2&type=Hive

    **response**

    .. sourcecode:: http

        HTTP/1.1 200 OK
        Content-Type: application/json

    .. sourcecode:: json

        {
            "job_types": [
                {
                    "name": "Hive",
                    "plugins": [
                        {
                            "description": "The Hortonworks Sahara plugin.",
                            "name": "hdp",
                            "title": "Hortonworks Data Platform",
                            "versions": {
                                "1.3.2": {
                                    "job_config": {
                                        "args": {},
                                        "configs": [
                                            {
                                                "description": "Reduce tasks.",
                                                "name": "mapred.reduce.tasks",
                                                "value": "-1"
                                            }
                                        ],
                                        "params": {}
                                    }
                                }
                            }
                        }
                    ]
                }
            ]
        }


This is an abbreviated example that shows imaginary config hints.


Other end user impact
---------------------

The python-saharaclient should be extended to support this as well:

.. code::

  $ sahara job-types-list [--type] [--plugin [--plugin-version]]

Output should look like this (not sure where else to specify this):

.. code::

   +---------------------+-----------------------------------+
   | name                | plugin(versions)                  |
   +---------------------+-----------------------------------+
   | Hive                | vanilla(1.2.1), hdp(1.3.2, 2.0.6) |
   | Java                | vanilla(1.2.1), hdp(1.3.2, 2.0.6) |
   | MapReduce           | vanilla(1.2.1), hdp(1.3.2, 2.0.6) |
   | MapReduce.Streaming | vanilla(1.2.1), hdp(1.3.2, 2.0.6) |
   | Pig                 | vanilla(1.2.1), hdp(1.3.2, 2.0.6) |
   +---------------------+-----------------------------------+


Since config hints can return so much information, and description
fields for instance can contain so much text, how to support
config hints through the python-saharaclient is TBD.

As noted above, the `Plugin SPI` will be extended with optional
methods. Existing plugins that support EDP will be modified as
part of this change.

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

The UI will be able to take advantage of this information
and filter the job types available to the user on the forms.
It will also be able to make use of config hints.


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  tmckay

Other contributors:
  none

Work Items
----------

* Add basic endpoint support with optional methods in the plugin SPI

* Implement the methods for each plugin that supports EDP
    This can be done as a series of separate small CRs

* Add support to python-saharaclient
* Update documentation

Dependencies
============

None


Testing
=======

* Unit tests
* Tempest tests for API


Documentation Impact
====================

It should be added to the REST API doc.


References
==========

