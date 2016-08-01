..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

============================
Support for Boot from Volume
============================

This specification proposes to add boot from volume capability to Sahara.

Problem description
===================

Sahara engine provisions VMs using a Glance image directly. In most
installations this means that the image is copied as a local file the
Nova-compute host and used as a root disk for the VM.

Having the root disk as a plain file introduces some limitations:

* Nova VM live migration (or Host evacuation) is not possible.
* Root disk performance may significantly suffer if QCOW2 format is used.

Having root disk replaced with a bootable Volume backed by a distributed
backend or local disk storage will solve the limitation listed above.

Proposed change
===============

The ability to boot from volume still requires an image to serve as source.
This means that Volume base provisioning will require 2 steps.

* Create a bootable volume from a registered Sahara image.
* Boot a VM with the block_device_mapping parameter pointing to the created
  volume.

Volume based provisioning requires the volume size to be set explicitly. This
means that Sahara should take care about this parameter. The proposed way to
set the bootable volume size is take the root disk size of the flavor being
used for the VM.

If the user selects the Node Group to be provisioned from the volume, he should
set the boot_from_volume flag to True.

The volume based provisioning is different from image based and implies the
following change.

The image parameter from the instance template should be removed.

The instance Heat template should have a new section with the block device
mapping.


.. code-block:: yaml

  block_device_mapping: [{
   device_name: "vda",
   volume_id : {
     get_resource : bootable_volume },
   delete_on_termination : "true" }
  ]

The resource group definition should have a volume added by the following
template:

.. code-block:: yaml

  bootable_volume:
    type: OS::Cinder::Volume
    properties:
      size: <size derived from the flavor>
      image: <regular Sahara image>


Alternatives
------------

Alternatively the user may be allowed to chose an existing volume to boot from.
This however cannot guarantee that the provided volume is suitable for the
cluster installation. Sahara also requires a username metadata to be able to
log into VM. This metadata is only stored in images right now.

Data model impact
-----------------

Node Group Template, Node Group and Templates relation objects should now
have a boolean boot_from_volume field.

REST API impact
---------------

Convert and Boot from Volume flag should be added to all endpoints responsible
for Node Group manipulation.

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

An option should be added to the Node Group create and update forms.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  Nikita Konovalov nkonovalov@mirantis.com

Work Items
----------

* Implement boot from volume support at the backend with the db migration and
  Heat templates update.
* Add boot_from_volume flag support in python-saharaclient
* Add boot_from_volume flag support in sahara-dashboard

Dependencies
============

None

Testing
=======

* Unit test coverage in sahara and python-saharaclient repositories.
* Integration test coverage in sahara-tests framework.

Documentation Impact
====================

* REST API documents should be updated.
* General user documentation should describe the behavior introduced by the
  boot_from_volume flag.

References
==========

None
