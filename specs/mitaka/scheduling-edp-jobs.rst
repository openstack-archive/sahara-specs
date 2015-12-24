..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=======================================================
Add ability of scheduling EDP jobs for sahara
=======================================================

https://blueprints.launchpad.net/sahara/+spec/enable-scheduled-edp-jobs

This spec is to allow running scheduled edp jobs in sahara.

Problem description
====================

Currently sahara only supports one time click edp job.
But in many use cases, we need scheduled edp jobs. So by
adding scheduled ability to sahara edp engine, we can have
different implementation for different engine.(oozie,spark,storm etc)

Proposed change
===============

Define run_scheduled_job() interface in sahara base edp engine, then
implement this interface to the oozie engine. (Spark and storm engine
will be drafted in later spec)

Define two job execution types which indicate the job execution.
But we need not to change the API, instead we can add parameters into
job configs. Sahara will run different type of jobs according
to the job execution type. In the  api request, user should pass the
job_execution_type into job_configs.

Two job execution types:
(1)basic. runs simple one-time edp jobs, current sahara implementation
(2)scheduled. runs scheduled edp jobs

Example of a scheduled edp job request

POST /v1.1/{tenant_id}/jobs/<job_id>/execute

.. sourcecode::json

    {
        "cluster_id": "776e441b-5816-4d47-9e07-7ded58f9a5f6",
        "input_id": "af7dc864-6331-4c30-80f5-63d74b667eaf",
        "output_id": "b63780f3-13d7-4286-b731-88270fb204de",
        "job_configs": {
            "configs": {
                "mapred.map.tasks": "1",
                "mapred.reduce.tasks": "1"
            },
            "args": ["arg1", "arg2"],
            "params": {
                "param2": "value2",
                "param1": "value1"
            },
            "job_execution_info": {
                "job_execution_type": "scheduled",
                "start": "2015-5-15T01:00Z"
            }
        }
    }

For oozie engine implementation of scheduled edp jobs, we have
changes as blow:

Before running the job, sahara will create a coordinator.xml to describe
the job, then upload it to the HDFS EDP job lib folder. With this file,
sahara call oozie client to submit this job, the job will be run at the
scheduled time, the job status will be shown as "PREP" in the Horizon
page. Certainly, user can delete this job in preparing status as welll as
in running status.

Example of coordinator.xml

.. sourcecode::xml

    <coordinator-app name="job-name" frequency="${coord:minutes(5)}"
        start="${start}" end="${end}" timezone="UTC" xmlns="uri:oozie:
        coordinator:0.2">
        <action>
            <workflow>
                <app-path>${workflowAppUri}</app-path>
                <configuration>
                    <property>
                        <name>jobTracker</name>
                        <value>${jobTracker}</value>
                    </property>
                    <property>
                        <name>nameNode</name>
                        <value>${nameNode}</value>
                    </property>
                    <property>
                        <name>queueName</name>
                        <value>${queueName}</value>
                    </property>
                </configuration>
            </workflow>
        </action>
    </coordinator-app>

For spark and storm implementation, there is no implementation now, and we
will add them later.

Alternatives
------------

(1)Run edp job manually by login into the VM and running oozie command.
(2)users can create cron jobs

Data model impact
-----------------

None

REST API impact
---------------

There is no change here, and we can use current API,
POST /v1.1/{tenant_id}/jobs/<job_id>/execute
We can pass job_execution_type, start time, into job_configs
to sahara.

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

In Job launch page, add textbox for user to input start job time,
default value is now, to compatible with current implementation

Implementation
==============

Assignee(s)
-----------

Primary assignee:
   luhuichun(lu huichun)

Other contributors:
  None

Work Items
----------

* define scheduled job type
* create coordinator.xml before run job in edp engine
* upload the coordinator.xml to job's HDFS folder
* add run_schedule_job in oozie engine
* modify sahara api reference docs
* Add task to update the WADL at api-site

Dependencies
============

None.

Testing
=======

unit test in edp engine
add scenario integration test

Documentation Impact
====================

Need to be documented.

References
==========

oozie scheduled and recursive job implementation
https://oozie.apache.org/docs/4.0.0/CoordinatorFunctionalSpec.html
