..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=================================
Reduce Number Of Dashboard panels
=================================

https://blueprints.launchpad.net/sahara/+spec/reduce-number-of-panels

The sahara UI currently consists of 10 panels.  That is more than any other
Openstack service.  Additionally, the way that the panels are meant to
interact with each other is not intuitively obvious and can lead to confusion
for users.  The purpose of this spec is to propose reorganizing the sahara
UI into a more logical layout that will give both a cleaner look and a more
intuitive set of tools to the users.

Problem description
===================

The sahara dashboard has too many panels without enough context provided
to convey how each panel relates to the others.  They are currently grouped
vertically so that each appears close to the related panels in the list,
but there is no other visual cue given.  Given that, learning how to use the
dashboard can often be a frustrating exercise of trial and error.


Proposed change
===============

In the 10 panels that are currently in the dashboard, 4 of them
(Clusters, Cluster Templates, Node Group Templates, Image Registry) are
focused around cluster creation.  Given that, it makes sense to fold them
into a single panel which will be called "Clusters".  This will make it very
obvious where to go for all things cluster-related.  Each of the current
cluster-centric panels will have their own tab within the newly created panel.
There are 5 panels (Jobs, Job Templates, Job Binaries, Data Sources and
Plugins) that are focused around job creation and execution.

In addition to the conversion from panels to tabs, I'm proposing that the code
also gets reorganized to reflect the new layout (ie: moving code to be a
subdir of the panel that it will reside in -- there will be a top level
"clusters" panel that will contain subdirectories for each of the tabs in that
panel (clusters, cluster templates, node group templates, image registry).
That reorganization will result in the changing of the URL definitions and
will also change how we reference template names (Instead of
"project/data_processing.nodegroup-templates/whatever.html", the "/project"
part can be dropped, resulting in
"data_processing.nodegroup-templates/whatever.html", which is a slight
improvement in brevity.  Additionally, we can also elect to drop the
"data_processing." portion by renaming the template folders.  The oddity that
forced us to use that as a workaround has since been solved in the horizon
"enabled" mechanism.

Here is a quick ascii diagram showing the proposed UI layout:
+------------+---------------------------------------------------+
|            |                                                   |
| Clusters   | Clusters|Cluster Templates|Node Group Templates   |
| Jobs       | +--------------------------------------------+    |
|            | |                                            |    |
|            | |  Usual table for chosen object             |    |
|            | |                                            |    |
|            | |                                            |    |
|            | |                                            |    |
|            | |                                            |    |
|            | |                                            |    |
|            | |                                            |    |
|            | |                                            |    |
|            | +--------------------------------------------+    |
|            |                                                   |
+------------+---------------------------------------------------+

Addionally, the current "Guide" panel will be split to fit more readily
into the job and cluster spaces.  It currently occupies a separate tab and is
linked to from buttons in the other panels.  The buttons will remain the same,
but the code will be moved into subdirectories under clusters for the clusters
guide and jobs for the job guide.

Alternatives
------------

We could choose to leave the current panels alone and rely on stronger,
more verbose documentation and possibly provide links to that documentation
from within the dashboard itself.  After multiple discussions with the sahara
team, it is clear that this approach is not as desirable as reorganizing
the layout.

We could possibly avoid some of the code reorganization, but in my opinion,
it is important to have the code accurately reflect the layout.  It makes it
easier to design future changes and easier to debug.

Data model impact
-----------------

No data model impact

REST API impact
---------------

No REST API impact

Other end user impact
---------------------

End users will see a different layout in the sahara dashboard.  While the
difference in moving to tabs is significant, once an experienced user gets
into the tabs, the functionality there will be unchanged from how it
currently works.  Some of the URLs used will also change a bit to remove
any ambiguity (for instance the "details" URLs for each object type will be
renamed to become something like "ds-details" for the data source details
URLs).

Deployer impact
---------------

No deployer impact

Developer impact
----------------

No developer impact

Sahara-image-elements impact
----------------------------

No sahara-image-elements impact

Sahara-dashboard / Horizon impact
---------------------------------

There will be changes to the "enabled" files in the sahara-dashboard project,
but the process for installation and running will remain the same.  If you
are upgrading an existing installation, there will be some "enabled" files
that are no longer required (in addition to updated files).  We will need to be
sure to document (and script wherever possible) which files should be removed.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  croberts

Other contributors:
  None

Work Items
----------

Reorganize sahara UI code to convert multiple panels into 2 panels that
contain multiple tabs.  This will likely be spread into multiple patches
in order to try to minimize the review effort of each chunk.  Each chunk
is still going to be somewhat large though since it will involve moving
entire directories of code around.

Dependencies
============

None

Testing
=======

Each of the existing unit tests will still apply.  They will all need to be
passing in order to accept this change.  It is likely that the (few) existing
integration tests will need to be rewritten or adapted to expect the proposed
set of changes.


Documentation Impact
====================

The sahara-dashboard documentation is written and maintained by the sahara
team.  There will be several documentation updates required as a result of
the changes described in this spec.


References
==========

None
