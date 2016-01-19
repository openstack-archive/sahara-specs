..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==================================================================
Add an option to sahara-image-create to generate bare metal images
==================================================================

https://blueprints.launchpad.net/sahara/+spec/sahara-bare-metal-images

Bare metal image generation is supported by disk-image-create but there
is no option exposed in sahara-image-create that allows the generation
of sahara images for bare metal. If sahara is to support bare metal
deployments, we must have sahara bare metal images.

Problem description
===================

Users should have a simple option to generate a sahara bare metal image.
This option should be applicable to all platforms and all plugins. The
default behavior of sahara-image-create should remain unchanged -- it
should generate VM images if the option is not enabled.

To generate a sahara bare metal image, the "vm" element needs to be left
out of the element list and the "grub2", "baremetal", and
"dhcp-all-interfaces" elements should be added.

Proposed change
===============

Add a "-b" command line option that sets a boolean flag indicating bare
metal image generation. If this option is set, add
"grub2 baremetal dhcp-all-interfaces" to the list of elements
passed to disk-image-create and prevent the "vm" element from being passed.

Do not bother making the list of baremetal elements modifiable from
the shell. It's unlikely that capability will be needed.

If the "-b" command line option is not set, no bare metal elements
will be added to the element list and the "vm" element will not be
removed.

Alternatives
------------

None


Data model impact
-----------------

None

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

No impact beyond the change to sahara-image-create itself.

Sahara-dashboard / Horizon impact
---------------------------------

None

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  tmckay

Work Items
----------

A single patch to diskimage-create.sh

Dependencies
============

None

Testing
=======

Manually test the generation of bare metal images for all OSs and plugins.
Image generation should succeed and generate .vmlinuz, .initrd, and .qcow2
files. However, successful generation doesn't guarantee that they will
actually work (for instance in Kilo, Fedora and CentOS 6 images will not
boot correctly in ironic)

For this change, it is enough that the image generation completes.

Documentation Impact
====================

The "-b" option should be mentioned where we discuss image generation.

References
==========

None
