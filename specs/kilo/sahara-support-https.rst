..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

============================
HTTPS support for sahara-api
============================

https://blueprints.launchpad.net/sahara/+spec/sahara-support-https

Most OpenStack services support running server supporting HTTPS connections.
Sahara must support such way too.

Problem description
===================

There are two common ways to enable HTTPS for the OpenStack service:

1. TLS proxy. Proxy communicates with user via HTTPS and redirects all
   requests to service via unsecured HTTP. Keystone is configured to point
   on HTTPS port. Internal port is usually closed for outside using firewall.
   No additional work to support HTTPS required on service side.

2. Native support. Service can be configured to expect HTTPS connections. In
   this case service handles all security aspects by itself.

Most OpenStack services support both types of enabling SSL. Sahara currently
can be secured only using TLS proxy.

Proposed change
===============

Add ability to Sahara API to listen on HTTPS port.

Currently there is no unified way for OpenStack services to work with HTTPS.
Process of unification started with sslutils module in oslo-incubator. Sahara
could use this module to be on the same page with other services.

Alternatives
------------

Copy-paste SSL-related code from other OpenStack project.

Data model impact
-----------------

None

REST API impact
---------------

None

Other end user impact
---------------------

* python-saharaclient should support SSL-related options

Deployer impact
---------------

One more option to consider.

Developer impact
----------------

None

Sahara-image-elements impact
----------------------------

None

Sahara-dashboard / Horizon impact
---------------------------------

Add SSL-related parameters to pass to python client.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  alazarev (Andrew Lazarev)

Other contributors:
  None

Work Items
----------

* Implement feature in Sahara

  * Import sslutils
  * Configure WSGI server to HTTPS

* Add support to python-saharaclient
* Add support to devstack
* Add documentation

Dependencies
============

* sslutils module from oslo-incubator


Testing
=======

Devstack doesn't have HTTPS testing for now. It looks like manual testing is
the only option.

Documentation Impact
====================

Need to be documented.

References
==========

None