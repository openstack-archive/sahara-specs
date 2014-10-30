..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=================================================================
Enable Swift resident Hive tables for EDP with the vanilla plugin
=================================================================

https://blueprints.launchpad.net/sahara/+spec/edp-hive-vanilla-swift

vanilla1 plugin supports Hive but Hive can't access Swift.
vanilla2 plugin not supports Hive.
This proposal aims that Hive can process the table that stored in Swift
and add hive support to vanilla2 plugin.


Problem description
===================

When Hive processes table that stored in Swift, `hiveserver` has to
access Swift. But `hiveserver` cannot read SwiftFS configuration from
Hive query. `hiveserver` reads configurations from xml files only
(core-site.xml/hive-site.xml).
Therefore `hiveserver` doesn't know the authentication info and can't access
Swift.

In vanilla2 plugin, doesn't implemented hive support code.

Proposed change
===============

`hiveserver` reads configuration at startup time only and doesn't read
configuration by hive query. Therefore sahara have to pass the swift
authentication info to `hiveserver` through hive-site.xml before
launching `hiveserver`.

When Hive enabled cluster created, Sahara creates keystone TRUST and Swift
proxy user (cluster-scope). And Sahara writes swift proxy user and TRUST to
hive-site.xml before `hiveserver` started.

This will enable `hiveserver` to read a authentication info at startup-time.
And when hive query arrived, `hiveserver` can access the swift with
cluster-scoped TRUST.

When cluster terminated, Sahara removes TRUST and proxy user before cluster's
database entry removed. If error on removing a TRUST, cluster goes error status
and not removed. Therefore there is no orphaned proxy user.

In vanilla2 plugin, implement hive support code in reference to vanilla1.

Alternatives
------------

1. Adds auth config field (`fs.swift.service.sahara.username`,
`fs.swift.service.sahara.password`) to hive-default.xml.
And end-user inputs username/password for cluster configurations.
I think this alternative is very inconvenience.

2. Fix hive-server to read a configuration from hive query.


Data model impact
-----------------

Database schema: No change

Cluster config: Change. But no affects others
  cluster-config is dict. Adds internal key and stores swift proxy user info
  and TRUST-id. This key doesn't send to client.


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
  k.oikw (Kazuki OIKAWA)

Other contributors:
  None


Work Items
----------

* Implement proxy user / TRUST creation feature when cluster creates
* Implementation of sanitizing cluster-config
* Implementation of writing proxy user and TRUST to hive-site.xml
* Implement hive support code in vanilla2


Dependencies
============

* https://blueprints.launchpad.net/sahara/+spec/edp-swift-trust-authentication


Testing
=======

We will add a integration test. This test checks whether Hive can process the
table that stored in the swift executes successfully.


Documentation Impact
====================

None


References
==========

None
