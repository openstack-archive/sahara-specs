..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==================================================
Make anti affinity working via server groups
==================================================

https://blueprints.launchpad.net/sahara/+spec/anti-affinity-via-server-groups

Server groups is an openstack way to implement anti affinity. Anti affinity
was implemented in Sahara before server groups were introduced in nova. Now it
is time to replace custom solution with the common one.


Problem description
===================

Direct engine uses manual scheduler hints for anti affinity.

Heat engine has limited anti affinity support and also uses scheduler hints
(https://bugs.launchpad.net/sahara/+bug/1268610).

Nova has generic mechanism for this purpose.

Proposed change
===============

Proposed solution is to switch both engines to implementation that uses server
groups.

Current server group implementation has limitation that each server could
belong to a single server group only. We can handle this constraint by having
one server group per cluster. In this case each instance with affected
processes will be included to this server group. So, with such implementation
there will be no several affected instances on the same host even if they
don't have common processes. Such implementation is fully compliant with all
documentation we have about anti affinity.

We need to keep backward compatibility for direct engine. Users should be
able to scale clusters deployed on Icehouse release. Sahara should update
already spawned VMs accordingly. Proposed solution - Sahara should check
server group existence during scaling and update whole cluster if server group
is missing.

We don't care about backward compatibility for heat engine. It was in beta
state for Icehouse and there are other changes that break it.

Alternatives
------------

We can implement anti affinity via server groups in heat engine only. We will
stop support of direct engine somewhere in the future. So, we can freeze
current behavior in direct engine and don't change it until it is deprecated
and removed.

Data model impact
-----------------

None

REST API impact
---------------

None

Other end user impact
---------------------

Anti affinity behavior changes in case of several involved processes. Before
the change there was possible situation when several instances with affected
(but different) processes are spawned on the same host. After the change all
instances with affected processes will be scheduled to different hosts.

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
  alazarev

Work Items
----------

Implement change

Dependencies
============

None

Testing
=======

Will be covered by current integration tests.

Documentation Impact
====================

Need note in upgrade notes about anti affinity behavior change.

References
==========

* Team meeting minutes: http://eavesdrop.openstack.org/meetings/sahara/2014/sahara.2014-08-21-18.02.html
