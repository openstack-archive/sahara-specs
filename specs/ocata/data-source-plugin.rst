==========================================
Data Source and Job Binary Pluggability
==========================================

https://blueprints.launchpad.net/sahara/+spec/data-source-plugin

Sahara allows multiple types of data source and job binary. However, there's
no clean abstraction around them, and the code to deal with them is often very
difficult to read and modify. This change proposes to create clean
abstractions that each data source type and job binary type can implement
differently depending on its own needs.

Problem description
===================

Currently, the data source and job binary code are spread over different
folders and files in Sahara, making this code hard to change and to extend.
Right now, a developer who wants to create a new data source needs to look all
over the code and modify things in a lot of places (and it's almost impossible
to know all of them without deep experience with the code). Once this change
is complete, developers will be able to create code in a single directory and
will be able to write their data source by implementing an abstract class.
This will allow users to enable data sources that they write themselves
(and hopefully contribute upstream) much more easily, and it will allow
operators to disable data sources that their own stack does not support
as well.

Proposed change
===============

This change proposes to create the data source and job binary abstractions as
plugins, in order to provide loading code dynamically, with a well defined
interface. The existing types of data sources and job binaries will be
refactored.

The interfaces that will be implemented are described below:

*Data Source Interface*

* prepare_cluster(cluster, kwargs) - Makes a cluster ready to use this
  data source. Different implementations for each data source: for Manila
  it will be mount the share, for Swift verify credentials, etc.

* construct_url(url, job_exec_id) - Resolves placeholders in data source URL.

* get_urls(cluster, url, job_exec_id) - Returns the data source url and the
  runtime url the data source must be referenced as. Returns: a tuple of the
  form (url, runtime_url).

* get_runtime_url(url, cluster) - If needed it will construct a runtime
  url for the data source, by the default if a runtime url is not needed
  it will return the native url.

* validate(data) - Checks whether or not the data passed through the API
  to create or update a data source is valid.

* _validate_url(url) - This method is optional and can be used by the
  validate method in order to check whether or not the data source url
  is valid.

*Job Binary Interface*

* prepare_cluster(cluster, kwargs) - Makes a cluster ready to use this
  job binary. Different implementations for each data source: for Manila it
  will be mount the share, for Swift verify credentials, etc.

* copy_binary_to_cluster(job_binary, cluster) - If necessary, pull
  binary data from the binary store and copy that data to a useful path
  on the cluster.

* get_local_path(cluster, job_binary) - Returns the path on the local
  cluster for the binary.

* validate(data) - Checks whether or not the data passed through the API
  to create or update a job binary is valid.

* _validate_url(url) - This method is optional and can be used by the
  validate method in order to check whether or not the job binary url
  is valid

* validate_job_location_format(entry) - Pre checks whether or not the API
  entry is valid.

* get_raw_data(job_binary, kwargs) - Used only by the API, it returns
  the raw binary. If the type doesn't support this operation it should
  raise NotImplementedException.

These interfaces will be organized in the following folders structure:

* services/edp/data_sources - Will contain the data source interface and
  the data source types implementations.

* services/edp/job_binaries - Will contain the job binary interface and
  the job binary types implementations.

* services/edp/utils - Will contain utility code that can be shared
  by data source implementations and job binary implementations.
  Per example: Manila data source implementation will probably share some
  code with the job binary implementation.

Probably some changes in the interface are possible until the changes are
implemented (parameters, method names, parameter names), but the main
structure and idea should stay the same.

Also a plugin manager will be needed to deal directly with the
different types of data sources and job binaries and to provide methods for
the operators to disable/enable data sources and job binaries dynamically.
This plugin manager was not detailed because is going to be similar to the
plugin manager already existent for the cluster plugins.

Alternatives
------------

A clear alternative is let things the way they are, but Sahara would be more
difficult to extend and to understand; An alternative for the abstractions
defined in the Proposed Change section would be to have only one abstraction
instead of two interfaces for data sources and job binaries since these
interfaces have a lot in common, implementing this alternative would remove
the edp/service/utilities folder letting the code more unified and compact,
but job binary and data source code would be considered only one plugin,
which could difficult the pluggability feature of this change (per example:
the provider would not be able to disable manila for data sources, but enable
it for job binaries) and because of this it was not considered the best
approach, instead we keep job binaries and data sources apart, but in
contrast we need the utilities folder to avoid code replication.

Data model impact
-----------------

None

REST API impact
---------------

Probably some new methods to manage supported types of data sources and
job binaries will be needed (similar to the methods already offered by
plugins).

* data_sources_types_list() ; job_binaries_types_list()
* data_sources_types_get(type_name) ; job_binaries_types_get(type_name)
* data_sources_types_update(type_name, data) ;
  job_binaries_types_update(type_name, data)

Other end user impact
---------------------

None

Deployer impact
---------------

None

Developer impact
----------------

After this change is implemented developers will be able to add and enable
new data sources and job binaries easily, by just implementing the abstraction.

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

Primary assignee: mariannelinharesm

Other contributors: egafford

Work Items
----------

* Creation of a Plugin for Data Sources containing the Data Source Abstraction
* Creation of a Plugin for Job Binaries containing the Job Binary Abstraction
* HDFS Plugin Intern
* HDFS Plugin Extern
* Swift Plugin
* Manila Plugin
* Allow job engines to declare which data sources/job binaries they are
  capable of using (this may be needed or not depending if exists a job type
  that does not support a particular data source or job binary type)
* Changes in the API

Dependencies
============

None

Testing
=======

This change will require only changes in existing unit tests.

Documentation Impact
====================

Will be necessary to add a devref doc about the abstractions created.

References
==========

None
