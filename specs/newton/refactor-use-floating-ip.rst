..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=========================================================================
Refactor the logic around use of floating ips in node groups and clusters
=========================================================================

https://blueprints.launchpad.net/sahara/+spec/refactor-use-floating-ip

Currently, there is a boolean in the configuration file called
*use_floating_ips* which conflates the logic around the existence of
floating ips for instances and the use of floating ips for management by
sahara. This logic should be refactored.

Problem description
===================

The *use_floating_ips* boolean when set True has the following implications:

* All instances must have a floating ip, except for the case of using a
  proxy node.
* Every node group therefore must supply a floating ip pool value when using
  neutron, and nova must be configured to auto-assign floating ips when using
  nova networking.
* Sahara must use the floating ip for management.
* These requirements are in force across the application for every user,
  every node group, and every cluster.

When *use_floating_ips* is False and neutron is used:

* Any node group template that has a floating ip pool value set is not
  usable and will fail to launch. Again, there is an exception for this when
  using proxy nodes.
* Therefore simply modifying the value of *use_floating_ips* in the sahara
  configuration file will invalidate whole groups of existing, functioning
  templates.

As we move toward a world where virtual and baremetal clusters co-exist, we
need to modify the logic around floating ips to provide more flexibility. For
instance, a baremetal cluster that uses a flat physical network with no
floating ips should be able to co-exist in sahara with a VM cluster that
uses a virtual network with floating ips (currently this is not possible).

As nova, neutron, and ironic make further advances toward hybrid
networking and instance scenarios, sahara needs the flexibility to adapt to new
features. The current conflated logic is a roadblock to this flexibility.

Proposed change
===============

Logically the change is very simple, but implementation will require careful
analysis and testing since knowledge of floating ips and the use of nova vs
neutron networking is spread throughout the code base. This becomes
particularly complex in the logic around ssh connections and proxy gateways
and in the quota logic.

The proposed logical change goes like this:

* The semantics of *use_floating_ips* will be changed to mean "if an instance
  has a floating ip assigned to it, sahara will use that floating ip for
  management, otherwise it will use the internal ip". This flag will impose no
  other requirements, and no other expectations should be associated with it.

* In the case of neutron networking, if a node group specifies a floating ip
  pool then the instances in that node group will have floating ips. If the
  node group does not specify a floating ip pool this will not be an error but
  the instances will not have floating ips.

* In the case of nova networking, if nova is configured to auto-assign
  floating ips to instances then instances will have floating ips. If nova
  does not assign a floating ip, the instances will not have floating ips.

* In the case of neutron networking, *use_namespaces* set to True will
  continue to mean "use ip netns to connect to machines that do not have
  floating ips" but each instance will have to be checked individually to
  determine if it has a floating ip assigned. It will no longer be valid
  to test *CONF.use_namespases and not CONF.use_floating_ips*

Alternatives
------------

None. This logic has been in sahara since the beginning when nova networking
was exclusively used and things were simpler; it's time to refactor.

Data model impact
-----------------

None

REST API impact
---------------

None, although changes are needed in the validation code. The API itself
does not change, but what is semantically allowable does change (ie, whether
or not a node group *must* have a floating ip pool value)


Other end user impact
---------------------

Users will need to be educated on the shifting implications of setting
*use_floating_ips* and the choice they have when configuring node groups.

Deployer impact
---------------

This change *should be* transparent to existing instances of sahara. Templates
that are functioning should continue to function, and clusters that are
running should not be affected. What will change is the ability to control
use of floating ips in new clusters.

Developer impact
----------------

None

Sahara-image-elements impact
----------------------------

None

Sahara-dashboard / Horizon impact
---------------------------------

Uncertain. In the past there were configuration settings for the sahara
dashboard that touched on floating ip use. The current configuration
parameters should be reviewed to see if this holds any implication for horizon.

Implementation
==============

Assignee(s)
-----------

Trevor McKay has produced a patch for this which should be fairly complete,
but he is unable to finish it. He was in the testing phase when work on this
stopped, with fair confidence that the solution works in neutron (but more
testing needs to be done in a nova networking environment)

Primary assignee: tellesnobrega

Other contributors: tmckay

Work Items
----------

* Refactor floating-ip use
* Implement tests

Dependencies
============

None

Testing
=======

Unit tests are sufficient to cover changes to template validation routines
and logical flow in sahara (is sahara in a particular case trying to use a
floating ip or not?)

Scenario tests should be constructed for both values of *use_floating_ips*,
for both neutron and nova networking configurations, and for node groups
with and without floating ip pool values.

Documentation Impact
====================

The new implications of *use_floating_ips* should be covered in the
documentation on configuration values and set up of nova in the
nova networking case.

It should also be noted in discussion of node group templates that
floating ip pool values are no longer required or disallowed based
on the value of *use_floating_ips*

As mentioned above, it's unclear whether anything needs to change in
sahara dashboard configuration values. If something does change, then
horizon docs should be changed accordingly.

References
==========

None
