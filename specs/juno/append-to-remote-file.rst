..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

================================
Append to a remote existing file
================================


https://blueprints.launchpad.net/sahara/+spec/append-to-remote-file

Sahara utils remote can only create a new file and write to it or replace a
line for a new one, but it can't append to an existing file. This bp aims to
implement this feature.

Problem description
===================

When managing remote files, sahara can only create new files and replace lines
from existing one. The feature to append to an existing file doesn't exist
and it is necessary.

Proposed change
===============

Implement this feature following the idea of the write_to_file method
The code is basically the same, the change will be the method of opening
the file. Write uses 'w' we need to use 'a'.


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

None

Sahara-dashboard / Horizon impact
---------------------------------

None

Implementation
==============

Assignee(s)
-----------

Primary assignee:

  * tellesmvn

Work Items
----------

The implementation is very basic, the idea is similar to the write_file,
the necessary change is to open the remote file in append mode.

Dependencies
============

None

Testing
=======

None for now.

Documentation Impact
====================

None

References
==========

None
