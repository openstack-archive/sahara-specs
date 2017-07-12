..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==============================
Deprecation of CentOS 6 images
==============================

https://blueprints.launchpad.net/sahara/+spec/deprecate-centos6-images

Starting from the Newton release, the images based on CentOS 6
are available also with CentOS 7, sometimes even with more choices for
CentOS 7 version (CDH).
This spec propose to deprecate and remove the support for CentOS 6
images.


Problem description
===================

Keeping the support for CentOS 6 is increasinglly difficult.

The cloud image provided by the CentOS team can not be used as it is
(lack of resize) so a special image should be prepared. This is
`documented <http://git.openstack.org/cgit/openstack/sahara-image-elements/tree/diskimage-create/Create_CentOS_cloud_image.rst?h=6.0.1>`_
but the default image, which should be manually regenerated, is hosted on
sahara-files.mirantis.com, which will be discontinued.

Also, diskimage-builder's support for CentOS 6 is not so effective
as it should be, as most of the focus is (rightfully) on CentOS 7.

Example of issues which requires a workaround:

* https://bugs.launchpad.net/diskimage-builder/+bug/1534387
* https://bugs.launchpad.net/diskimage-builder/+bug/1477179

A blocker bug right now is:

* https://bugs.launchpad.net/diskimage-builder/+bug/1698551

The (non-blocking, even if they should be blocking) gate jobs for
sahara-image-builder fails due to the latter.


Proposed change
===============

The support for CentOS 6 images should be deprecated starting from Pike and
removed as soon as the compliance with the follows-standard-deprecation
allows us to do it.

The change mainly affects sahara-image-elements. The CentOS 6 would not be
built anymore by default while building all the images for a certain plugin
and a warning message would be printed if one of them is selected.

The code path which checks for CentOS 6 in Sahara services should be kept
as they are and not changed as long as the features is available even if
deprecated; after the removal the code can be restructured, if needed,
to not consider the CentOS 6 use case.

Alternatives
------------

Keep CentOS 6 support until it is retired officially (November 30, 2020)
or until diskimage-builder removes the support, but make sure that the
current issues are fixed. A change is needed anyway in the
sahara-image-elements jobs, as the building fails right now.

Data model impact
-----------------

None

REST API impact
---------------

None

Other end user impact
---------------------

Users won't be able to use CentOS 6 as base.

Deployer impact
---------------

None

Developer impact
----------------

None

Sahara-image-elements impact
----------------------------

Most of the described changes are in sahara-image-elements (see above).

Sahara-dashboard / Horizon impact
---------------------------------

Minor: remove the reference to CentOS 6 and the default cloud image
from the image registration panel when the feature is removed.


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  ltoscano


Work Items
----------

- do not build CentOS 6 image by default for a certain plugin
- add a warning message if one of them is requested
- inform the operators (openstack-operators@) about the change to evaluate
  the time for the removal


Dependencies
============

None


Testing
=======

If the change is implemented, the existing jobs for sahara-image-elements
will only test the supported images and won't fail.


Documentation Impact
====================

Add or change the list of supported base images.


References
==========

None
