..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=====================
Designate Integration
=====================

https://blueprints.launchpad.net/sahara/+spec/designate-integration

Designate provides DNS as a service so we can use hostnames instead of
IP addresses. This spec is proposal to implement this feature in Sahara.

Problem description
===================

Users want to use meaningful hostnames instead of just ip addresses. Currently
Sahara only changes ``/etc/hosts`` files when deploying the cluster and
it allows to resolve vms by hostnames only through console and only between
these vms. But with Designate integration the hostnames of vms could be used in
the dashboard and we don't need to change ``/etc/hosts``.

Proposed change
===============

With this change we'll be able to act in such way: user have pre installed
Designate on controller machine and set up it with network (see [0]). Also user
have added DNS address to ``/etc/resolv.conf`` file on client machine. User
creates a cluster template in which it chooses domain names for internal and
external resolution. Then user launches the cluster and all its instances can
be resolved by their hostnames:

  1. from user client machine (e.g., links in Sahara-dashboard)
  2. between instances

Designate integration will work if we point that we want to use it in sahara
configs. So new config option should be added in the ``default`` section:

  * ``use_designate``: boolean variable which indicates should Sahara use
    Designate or not (by default is False).
  * ``nameservers``: list of servers' ips with pre installed Designate. This
    is required if 'use_designate' option is True.

Domain records format will be ``instance_name.domain_name.``. Domain records
will be created by Heat on create heat stack step of cluster creation process.
The hostnames collision isn't expected: 1. Designate allows only unique domain
names so user can't create domain with the same name in different tenants. Also
domain names are created in one tenant and aren't available in another.
2. Nova allows to launch instances with the same name but Sahara adds indexes
(0,1,2,...) for each instance name. The single collision case is when we launch
two clusters with the same node group names and the same cluster names then
Designate doesn't allow to create duplicated records so user can just change
cluster name.

We should maintain backward compatibility. Backward compatibility cases:

  * Old version of Openstack. Then user can switch off designate with
    ``use_designate = false`` (this value by default).
  * Cluster already exists. Then user can't use designate feature.

Addresses of the domain servers should be written to ``/etc/resolv.conf`` files
on each of vms in order to successfully resolve created domain records across
these vms. It could be done with cloud-init capabilities.

Alternatives
------------

None. We can leave all as is: now we change ``/etc/hosts`` files for resolving
hostnames between vms.

Data model impact
-----------------

Cluster, Cluster Template and Instance entities should have two new columns:

  * ``internal_domain_name``
  * ``external_domain_name``

REST API impact
---------------

None

Other end user impact
---------------------

User need to pre install and setup designate by self (see [0]). Also user
should change ``resolv.conf`` files on appropriate machines in order to
resolve server with designate.

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

Cluster template creation page will contain additional 'DNS' tab with two
dropdown fields: internal domain and external domain.

Implementation
==============

None

Assignee(s)
-----------

Primary assignee:
  msionkin (Michael Ionkin)

Work Items
----------

* implement designate heat template
* implement writing dns server address to ``resolv.conf`` files
* provide backward compatibility
* add new db fields
* add tab and fields for cluster template page in Sahara dashboard
* add unit tests

Dependencies
============

* python-designateclient for Sahara-dashboard

Testing
=======

Unit tests should be added.

Documentation Impact
====================

This feature should be documented.

References
==========

[0] http://docs.openstack.org/mitaka/networking-guide/adv-config-dns.html
