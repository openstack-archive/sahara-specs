..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

======================
Indirect access to VMs
======================

https://blueprints.launchpad.net/sahara/+spec/indirect-vm-access

This blueprint proposes one more way for Sahara to manage VMs. Management
could be done via VM that works as proxy node. In this case access
from controller should be given for one VM only.

Problem description
===================

Currently there are several ways to give Sahara access to VMs:

1. flat private network

   * not secure
   * doesn't work with neutron

2. floating IPs

   * all nodes need to have floating IPs

     - floating IPs are limited resource
     - floating IPs are usually for external world, not for access from
       controller

   * all nodes need to be accessible from controller nodes

     - it is more complicated in HA mode

   * access to data nodes should be secure

3. net_ns

   * hard to configure
   * can be inappropriate
   * doesn't work in HA mode

5. tenant-specific proxy node (https://review.openstack.org/#/c/131142/)

   * proxy setting is for the whole system (template based)
   * proxy can't be configured for a specific cluster
   * proxy node needs to be spawned and configured manually

4. agents

   * not implemented yet
   * require external message queue accessible from VMs and controllers
   * require maintenance of agents

So, there can be cases when none of listed approaches work.

Proposed change
===============

This blueprint proposes one more way to access VMs by Sahara.
Sahara will use one of spawned VMs as proxy node and gain access to all other
nodes through it. Access to VM that has proxy node role could be gained using
any of methods above.

Sahara will understand which node to use as a proxy node by "is_proxy" field
of nodegroup. If this nodegroup contains several instances the first one will
be used as proxy (this leaves space for load balancing).

So, proposed workflow:

1. Nodegoup object is extended with "is_proxy" field, horizon is changed
   accordingly.
2. User selects "is_proxy" checkbox for one of node groups (manager, master or
   separate one). If proxy is used for separate node group it could be with
   really small flavor.
3. Sahara spawns all infrastructure
4. Sahara communicates with all instances via node with proxy role. Internal
   IPs are used for communication. This removes restriction that all nodes
   must have floating IP in case if floating network used for management. In
   case with proxy node this restriction will be applied to proxy node only.

Pros:

1. we need external access to only one VM.
2. data nodes could be isolated

Cons:

1. Indirect access could be slower.
2. Loss of proxy means loss of access to entire cluster. Intellectual
   selection is possible, but not planned for the first implementation.

Implementation will extend global proxy implemented at
https://review.openstack.org/#/c/131142/. For indirect access Paramiko-based
analogy of "ssh proxy nc host port" command will be used. Paramiko
implementation will allow to use private keys from memory.

Note, proxy command can still be used to access proxy instance.

Implementation details
----------------------

Sahara uses two ways of access to instances:

1. SSH
2. HTTP

SSH access
++++++++++

For ssh access one more layer of ssh will be added. Old code:

.. sourcecode:: python

    _ssh.connect(host, username=username, pkey=private_key, sock=proxy)

New code:

.. sourcecode:: python

    _proxy_ssh = paramiko.SSHClient()
    _proxy_ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())

    _proxy_ssh.connect(proxy_host, username=proxy_username,
                       pkey=proxy_private_key, sock=proxy)

    chan = _proxy_ssh.get_transport().open_session()
    chan.exec_command("nc {0} {1}".format(host, SSH_PORT))

    _ssh.connect(host, username=username, pkey=private_key, sock=chan)

HTTP access
+++++++++++

Http access will be implemented in a similar way with ProxiedHTTPAdapter.
SshProxySocket class will be implemented that corresponds to netcat socket
running on remote host.

Note, if proxycommand present, it will be passed to paramiko directly without
involving NetcatSocket class.

Alternatives
------------

This blueprint offers one more way to access VMs. All existing ways will remain
unchanged.

Data model impact
-----------------

None

REST API impact
---------------

New boolean field "is_proxy" in nodegroup and nodegroup template objects.

Other end user impact
---------------------

None

Deployer impact
---------------

One more deployment option to consider.

Developer impact
----------------

None

Sahara-image-elements impact
----------------------------

None

Sahara-dashboard / Horizon impact
---------------------------------

Checkbox in nodegroup template edit form.

Implementation
==============

Assignee(s)
-----------

Primary Assignee:

Andrew Lazarev (alazarev)

Work Items
----------

* Sahara core changes
* Python client changes
* Horizon changes
* Doc changes

Dependencies
============

* Global proxy implementation (https://review.openstack.org/#/c/131142/)

Testing
=======

Manually

Documentation Impact
====================

The feature needs to be documented.

References
==========

None
