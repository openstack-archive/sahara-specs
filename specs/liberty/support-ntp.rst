..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=========================================
Support NTP service for cluster instances
=========================================

https://blueprints.launchpad.net/sahara/+spec/support-ntp

Sahara can support NTP (Network Time Protocol) for clusters instances.
NTP is intended to synchronize time on all clusters instances. It can
be useful for several services on Cloudera, HDP clusters (see [1] for
reference).

Problem description
===================

Sahara should support configuring NTP on cluster instances since it's
required to HDP and Cloudera clusters.

Proposed change
===============

First, it's proposed to preinstall ntp daemon on all images via
``sahara-image-elements``. For clean images will not configure
ntp at all. Also installing ntp on instances of long living clusters
can be prevented, as well.

As a second step we should add new common string config option to
sahara in ``general`` section of ``cluster configs``, that will allow
to specify own NTP server for current cluster.  This config option can
be supported in all plugins and will allow to install NTP on all
cluster instances with specified NTP server. As option, we can allow
disabling NTP, at least for fake plugin. So, following plugin options
we will have:

1. ``NTP_ENABLED``: default value is True, this option is required to allow
   disabling ntp on cluster instances
2. ``NTP_URL``: default value is empty string. So, if user input of this
   option is empty string, we will use default ntp server from
   ``sahara.conf``. Otherwise, user input will be used.

As the third step, we should provide new config for ``sahara.conf``
that will to specify default NTP server on current sahara installation.
It can useful because default NTP server can be different in different
regions. Also it would allow to use NTP server that was installed
specially for current lab and current sahara installation.

Second step require to have common options for all plugins to avoid
code duplication for all plugins. All options can be added to
``plugins/provisioning.py`` as well.

Alternatives
------------

We can store ``ntp_url`` and ``ntp_enabled`` as cluster column, it's looks like
long story for being merged: sahara-side code -> python-saharaclient ->
(long long story) horizon support.

Data model impact
-----------------

We don't need extra migrations, we will store information about NTP
server in ``general`` section of ``cluster configs``.

Storing NTP server in separate column in database not really useful,
since we can store this information just in cluster configs.

REST API impact
---------------

None

Other end user impact
---------------------

User will have ability to specify own NTP server for current cluster
and current sahara installation.

Deployer impact
---------------

None

Developer impact
----------------

None

Sahara-image-elements impact
----------------------------

It's required to preinstall NTP on all images.

Sahara-dashboard / Horizon impact
---------------------------------

Since Sahara already have ability to expose all general config options during
cluster-template creation, we don't need extra modifications on Horizon side.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  vgridnev

Other contributors:
  sreshetniak

Work Items
----------

Proposed change will contain following steps:

1. Install NTP on images in ``sahara-image-elements``.
2. Add ability to install NTP on cluster instances and add required
   config options.
3. Add documentation for feature.

Dependencies
============

Depends on Openstack requirements

Testing
=======

Feature will covered with integration tests as well.

Documentation Impact
====================

Need to document feature and all config options, which will be used for
NTP configuration.

References
==========

[1] http://www.cloudera.com/content/cloudera/en/documentation/core/latest/topics/install_cdh_enable_ntp.html
