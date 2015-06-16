..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=======================================================
Add recurrence EDP jobs for sahara
=======================================================

https://blueprints.launchpad.net/sahara/+spec/recurrence-edp-job

This spec is to allow running recurrence edp jobs in sahara.

Problem description
====================

Currently sahara only supports one time click edp job.
But sometimes, we need recurrence edp jobs. For example, from 8:00AM
to 8:00PM, run a job every 5mins. By adding recurrence edp jobs to
sahara edp engine, we can have different engine implementation.
(oozie,spark,storm etc)

Proposed change
===============

Define run_recurrence_job() interface in sahara base edp engine, then
implement this interface to the oozie engine. (Spark engine will be
drafted in later spec)

Add one job execution type named "recurrence". Sahara base job engine
will run different type of jobs according to the job execution type.

Add one sahara perodic task named update_recurrence_job_statuses() to
update the recurrence jobs's sub-jobs running status.

Add validation for invalid data input and check if the output url is
already exist.

Example of recurrence edp job request

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
                "job_execution_type": "recurrence",
                "start": "2015-5-15T08:00Z"
                "end": "2015-5-15T09:00Z"
                "period_minutes": 5
            }
        }
    }

For supporting this feature,add a new table named child_job_execution
which will store sub-jobs of the father recurrence job. When create a
recurrence edp job, in the same time, we create M(configured) sub-jobs,so
we can show up the status of the M sub-jobs. perodic task will check the
status of the M child jobs's status and update them, and also if there is
one child job is finished, we create a new feature-run child job to fill
the vacancy, in this case, we maintain a M-row-window in the DB, so this
will avoid the endless child job creation.if we delete this recurrence
job, we delete it's child jobs in the child_job_execution table,including
finished jobs and future-run jobs.

For creating the child job execution table,we just add one more column
named "father_job_id" based on job execution table which points to it's
father recurrence job.

For Horizon changes, considering we may have many child jobs, so we only
show the latest M sub jobs when user click the recurrence edp job, and
add two query datetime picker with a button for user to search the history
finished child jobs.

For oozie engine implementation of recurrence edp jobs. we have
changes as below:

Implement the run_recurrence_job in the base edp engine. call oozie client
to submit the recurrence edp job. add perodic task update recurrence job
statuses to update child jobs's status in the child_job_execution table.

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

For spark implementation, no implementation here, will add them later.

Alternatives
------------

Run edp job manually by login into the VM and running oozie command.

Data model impact
-----------------

add a new table named child job execution.Just add one more column named
father_job_id than the current job execution table. Other columns are
totally the same as job execution table.



REST API impact
---------------

There is no change here, we can use current API,
POST /v1.1/{tenant_id}/jobs/<job_id>/execute
We can pass job_execution_type, start time, end time, period_minutes into
job_configs.

::

    CREATE TABLE child_job_execution (
        id VARCHAR(36) NOT NULL,
        job_id VARCHAR(36),
        father_job_id VARCHAR(36),
        tenant_id VARCHAR(80),
        input_id VARCHAR(36),
        output_id VARCHAR(36),
        start_time DATETIME,
        end_time DATETIME,
        info TEXT,
        cluster_id VARCHAR(36),
        oozie_job_id VARCHAR(100),
        return_code VARCHAR(80),
        job_configs TEXT,
        extra TEXT,
        data_source_urls TEXT,
        PRIMARY KEY (id),
        FOREIGN KEY (job_id)
            REFERENCES jobs(id)
            ON DELETE CASCADE
    );

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

In Job launch page, add select choose box for user to choose the job
execution type, it has three values(basic, scheduled, recurrence),default
value is basic, which is the one-click running job.if user choose
recurrence, there will be two datetime picker named as start and end time
and a textbox for user to input the period_minutes.

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

* define recurrence job type
* add one perodic task named update_recurrence_job_statuses
* create coordinator.xml before run job in edp engine
* upload the coordinator.xml to job's HDFS folder
* add run_recurrence_job in sahara base engine
* modify sahara api reference docs
* add task to update the WADL at api-site

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
