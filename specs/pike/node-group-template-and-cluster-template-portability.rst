..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

====================================================
Node Group Template And Cluster Template Portability
====================================================

https://blueprints.launchpad.net/sahara/+spec/portable-node-group-and-cluster-templates

Sahara allows creation of node group templates and cluster templates. However,
when there’s need to create template with the same parameters on another
openstack deployment one must create new template rewriting parameters by
hand. This change proposes to create functions to export these templates to
JSON files and import them later.

Problem description
===================

Sahara provides the ability to create templates for node groups and clusters.
In the case where the user has access to multiple clouds or is constantly
redeploying development environment it is very time consuming to recreate all
the templates. We aim to allow the user to have the option to download a
template and also upload an existing template to Sahara for a quicker setup.

Proposed change
===============

This change proposes to allow import and export of cluster templates and node
group templates. Uses node_group_template_get(node_group_template_id) and
node_group_template_create(parameters) to connect to database.
Templates will be changed before export so that IDs and other sensitive
information is not exported. Some additional information will be requested for
import of a template.
REST API changes:
Node group template interface
* node_group_template_export(node_group_template_id) - exports node group
template to a JSON file
Cluster template interface
* cluster_template_export(cluster_template_id) - exports cluster template to a
JSON file
UI changes:
Node group template interface:
* field for exporting node group template
* field for importing node group template - uses ngt create
Cluser template interface:
* field for exporting cluster template
* field for importing cluster template - uses cluster create
CLI  changes:
dataprocessing node group template export
dataprocessing node group template import
dataprocessing cluster template export
dataprocessing cluster template import


Alternatives
------------

A clear alternative is let things be the way they are, but it makes Sahara
hard to configure and reconfigure if it was configured before.

Data model impact
-----------------

None

REST API impact
---------------

* node-group-templates/{NODE_GROUP_TEMPLATE_ID}/export
* cluster-template/{CLUSTER_TEMPLATE_ID}/export


Other end user impact
---------------------

Export and import of node group templates and cluster templates available in
both CLI and UI.

Deployer impact
---------------

Simplified deployment, deployer can download pre existing templates if needed.

Developer impact
----------------

None

Sahara-image-elements impact
----------------------------

None

Sahara-dashboard / Horizon impact
---------------------------------

An option to export and another to import a template will be added. An option
to import a template will have needed fields to complete the template.

Implementation
==============

Assignee(s)
-----------

* Iwona
* tellesmvn


Work Items
----------

* Add export of a node group template
* Add import of a node group template
* Add export of a cluster template
* Add import of a cluster template
* Testing
* Documentation


Dependencies
============

None

Testing
=======

Unit tests will be added.

Documentation Impact
====================

Documentation about new features will be added.


References
==========

None
