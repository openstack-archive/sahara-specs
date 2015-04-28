..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===================================
Unified Map to Define Job Interface
===================================

https://blueprints.launchpad.net/sahara/+spec/unified-job-interface-map

This specification proposes the addition of an "interface" map to the API for
job creation, such that the operator registering a job can define a unified,
human-readable way to pass in all arguments, parameters, and configurations
that the execution of that job may require or accept. This will allow
platform-agnostic wizarding at the job execution phase and allows users to
document use of their own jobs once in a persistent, standardized format.

Problem description
===================

At present, each of our data processing engines require or may optionally take
any of arguments, parameters, configuration values, and data sources (which
may be any of 0, 1, or many inputs to 0, 1, or many outputs). This forces
users to bear the burden of documenting their own jobs outside of Sahara,
potentially for job operators who may not be particularly technical.

A single, human readable way to define the interface of a job, that can be
registered at the time of job registration (rather than job execution,) would
allow several benefits:

* A more unified UI flow across plugins
* A clean separation of responsibility between the creator of the job (likely
  a technical user) and the executor of the job
* A means of correcting our current assumptions regarding data sources (where
  for several plugins we are inappropriately assuming 1 input and 1 output
  source)

Proposed change
===============

When creating a job, an optional "interface" list may be added to the job
json (though we are fundamentally creating a map, presenting a list structure
will allow more intuitive ordering and fewer unnecessary error cases.) Each
member of this list describes an argument to the job (whether it is passed
as a configuration value, a named argument, or a positional argument.)

The interface is described by the following jsonschema object field:

::

    "interface": {
        "type": "array",
        "uniqueItems": True,
        "items": {
            "type": "object",
            "properties": {
                "name": {
                    "type": "string",
                    "minLength": 1
                },
                "description": {
                    "type": "string"
                },
                "mapping": {
                    "type": "object",
                    "properties": {
                        "type": {
                            "type": "string",
                            "enum": ["args", "configs", "params"]
                        },
                        "location": {
                            "type": "string",
                            "minLength": 1
                        }
                    },
                    "additionalProperties": False,
                    "required": [
                        "type",
                        "location"
                    ]
                },
                "value_type": {
                    "type": "string",
                    "enum": ["string",
                             "number",
                             "data_source",
                             "input_data_source",
                             "output_data_source"],
                    "default": "string"
                },
                "required": {
                    "type": "boolean"
                },
                "default": {
                    "type": "string"
                }
            },
            "additionalProperties": False,
            "required": [
                "name",
                "mapping",
                "required"
            ]
        }
    }

Post-schema validations include:

1) Names must be unique.
2) Mapping is unique.
3) The set of all positional arguments' locations must be an unbroken integer
   sequence with an inclusive minimum of 0.
4) Positional arguments may not be required, but must be given default values
   if they are not.

The job execution will also have a simpler interface field definition,
described by:

::

    "interface": {
        "type": "simple_config"
    }

New error cases at execution time include:

1) One configuration value or parameter is given two definitions (one through
   the interface map and one via configs, params, or data sources.)
2) An interface value does not pass validation for the type specified for the
   field in question.
3) A key in the execution interface map does not equal any key in the job
   definition's interface map.
4) The specified mapping type is not accepted by the job type being created
   (for instance, specifying the params type for a Spark job.)
5) An input data source does not contain data.
6) An output data source contains data.

In the case of additional positional values, the positional arguments given
in the args list will be appended to the list of interface positional
arguments (whether provided or default values.) This will allow for an
``*args`` pattern, should a plugin permit it.

Params and configs passed via the current mechanism that do not overlap with
any key in the execution interface map will be merged and passed to the job as
normal. This also applies to $INPUT and $OUTPUT params passed via the input
source and output source fields.

Alternatives
------------

In truth, after discussion, it seems that there is not a good alternative to
the broad strokes of this plan (save doing nothing). Leaving all configuration
of jobs to the execution phase is a real difficulty given that our supported
data processing engines simply lack a unified interface. If we wish to create
a unified flow, we need to create one; if we want to create one, the job
definition phase produces the least user pain, and a simple, flat map is the
most legible and flexible thing that can do the job.

Data model impact
-----------------

A new table will need to be created for storage of interface fields, described
by the following DDL (rendered in MySQL syntax for friendliness):

::

    CREATE TABLE job_interface_arguments (
        id VARCHAR(36) NOT NULL,
        job_id VARCHAR(36) NOT NULL,
        name VARCHAR(80) NOT NULL, # ex: 'Main Class'
        description TEXT, # ex: 'The main Java class for this job.'
        mapping_type VARCHAR(80) NOT NULL, # ex: 'configs'
        location TEXT NOT NULL, # ex: 'edp.java.main_class'
        value_type VARCHAR(80) NOT NULL, # ex: 'string'
        required BOOL NOT NULL, # ex: 0
        order TINYINT NOT NULL, # ex: 1
        default_value TEXT, # ex: 'org.openstack.sahara.examples.WordCount'
        created_at DATETIME,
        PRIMARY KEY (id),
        FOREIGN KEY (job_id)
            REFERENCES jobs(id)
            ON DELETE CASCADE
    );

This table will have uniqueness constraints on (job_id, name) and (job_id,
mapping_type, location).

A new table will also need to be created for storage of execution fields,
described by:

::

    CREATE TABLE job_execution_arguments (
        id VARCHAR(36) NOT NULL,
        execution_id VARCHAR(36) NOT NULL,
        argument_id INT NOT NULL,
        value TEXT NOT NULL, # ex: 'org.openstack.sahara.examples.WordCount'
        created_at DATETIME,
        PRIMARY KEY (id),
        FOREIGN KEY (execution_id)
            REFERENCES job_executions(id)
            ON DELETE CASCADE,
        FOREIGN KEY (argument_id)
            REFERENCES job_interface_arguments(id)
            ON DELETE CASCADE
    );

This table will have a uniqueness constraint on (execution_id, argument_id).

Note: While the TEXT type fields above (save Description) could validly be
given an upper length limit and stored as VARCHARs, TEXT is safer in the case
that a job actually requires an overly long argument, or is configured with
a reasonably massive key. This implementation detail is certainly up for
debate re: efficiency vs. usability.

Happily, this change will not require a migration for extant data; the
interface fields table has a (0, 1, or many)-to-one relationship to the jobs
table, and the existing configs/params/args method of propagating job
execution data can continue to function.

REST API impact
---------------

The Create Job schema will have a new "interface" field, described above. Each
listed exceptional case above will generate a 400: Bad Request error.

This field will also be represented in all GET methods of the Job resource.

The Create Job Execution schema will have a new "interface" field, described
above. Each listed exceptional case above will generate a 400: Bad Request
error.

This field will also be represented in all GET methods of the Job Execution
resource.

No other impact is foreseen.

Note: I am profoundly open to better options for terminology throughout this
document. As "args", "params", and "configs" are already taken, naming of a
new option has become difficult. "Interface" and "Interface arguments" seem
to me to be the best option remaining in all cases. If you can do one better,
please do.

Note: As interface fields will be represented in the data layer as individual
records, it would be possible to create an entirely new set of CRUD methods
for this object. I believe that course of action to be unnecessarily heavy,
however: should the job binary change, the job must be recreated regardless,
and a sensible interface need not change for the life of any concrete binary.

Other end user impact
---------------------

python-saharaclient will require changes precisely parallel to the interface
changes described above.

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

The immediate change does not require a Horizon change. Any UI that utilizes
this feature should be represented as a separate blueprint and spec, and will
doubtless touch wizarding decisions which are wholly orthogonal to this
feature.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  egafford

Other contributors:
  None

Work Items
----------

1) API updates as specified.
2) DB layer updates as specified.
3) Job execution argument validation and propagation to clusters.
4) Testing.
5) Python-saharaclient updates and testing.

Dependencies
============

None at present.

Testing
=======

A tempest test will cover injection of each mapping type into jobs (args,
configs, params.) This will be tested via a Pig job, as that type may take all
of the above. This test will include arguments mapping to both a Swift
datasource and an HDFS datasource, to ensure that both URL types are preserved
through the flow.

Thorough unit testing is assumed.

Documentation Impact
====================

None that have not already been mentioned.

References
==========

Chat_ (2014/12/05; begins at 2014-12-05T16:07:55)

.. _Chat: http://eavesdrop.openstack.org/irclogs/%23openstack-sahara/%23openstack-sahara.2014-12-05.log
