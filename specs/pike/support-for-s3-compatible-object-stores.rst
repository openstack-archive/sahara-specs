..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Support for S3-compatible object stores
==========================================

https://blueprints.launchpad.net/sahara/+spec/sahara-support-s3

Following the efforts done to make data sources and job binaries "pluggable",
it should be feasible to introduce support for S3-compatible object stores.
This will be an additional alternative to the existing HDFS, Swift, MapR-FS,
and Manila storage options.

A previous spec regarding this topic existed around the time of Icehouse
release, but the work has been stagnant since then:
https://blueprints.launchpad.net/sahara/+spec/edp-data-source-from-s3

Problem description
===================

Hadoop already offers filesystem libraries with support for s3a:///path URIs,
so supporting S3-compatible object stores on Sahara is a reasonable feature
to add.

Within the world of OpenStack, many cloud operators choose Ceph RadosGW
instead of Swift. RadosGW object store supports access through either
the Swift or S3 APIs. Also, with some extra configuration, a "native"
install of Swift can also support the S3 API. For some users we may expect
the Hadoop S3 library to be preferable over the Hadoop Swift library as it
has recently received several enhancements including support for larger
objects and other performance improvements.

Additionally, some cloud users may wish to use other S3-compatible object
stores, including:

* Amazon S3 (including AWS Public Datasets)
* LeoFS
* Riak Cloud Storage
* Cloudian HyperStore
* Minio
* SwiftStack
* Eucalyptus

It is clear that adding support for S3 datasources will open up a new world of
Sahara use cases.

Proposed change
===============

An "s3" data source type will be added, via new code in
*sahara.service.edp.data_sources*.  We will need utilities to validate S3
URIs, as well as to handle job configs (access key, secret key, endpoint,
bucket URI).

Regarding EDP, there should not be much work to do outside of defining the new
data source type, since the Hadoop S3 library allows jobs to be run against S3
seamlessly.

Similar work will be done to enable an "s3" job binary type, including the
writing of "job binary retriever" code.

While the implementation of the abstraction is simple, a lot of work comes
from dashboard, saharaclient, documentation, and testing.

Alternatives
------------

Do not add support for S3 as a data source for EDP. Since the Hadoop S3
libraries are already included on the image regardless of this change,
users can run data processing jobs against S3 manually. We still may wish
to add the relevant JARs to the classpath as a courtesy to users.

Data model impact
-----------------

None

REST API impact
---------------

None (only "s3" as a valid data source type and job binary type in the schema)

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

On most images, hadoop-aws.jar needs to be added to the classpath. Generally
images with Hadoop (or related component) installed already have the JAR. The
work will probably take place during the transition from SIE to image packing,
so the work will probably need to be done in both places.

Sahara-dashboard / Horizon impact
---------------------------------

Data Source and Job Binary forms should support s3 type, with fields for
access key, secret key, S3 URI, and S3 endpoint. Note that this is a lot
of fields, in fact more than we have for Swift. There will probably be some
saharaclient impact too, because of this.


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  Jeremy Freudberg

Other contributors:
  None

Work Items
----------

* S3 as a data source
* S3 as a job binary
* Ensure presence of AWS JAR on images
* Dashboard and saharaclient work
* Scenario tests
* Documentation

Dependencies
============

None

Testing
=======

We will probably want scenario tests (although we don't have them for Manila).

Documentation Impact
====================

Nothing out of the ordinary, but important to keep in mind both user and
developer perspective.


References
==========

None
