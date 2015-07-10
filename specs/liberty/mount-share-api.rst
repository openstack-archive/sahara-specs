..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=========================================================
API to Mount and Unmount Manila Shares to Sahara Clusters
=========================================================

https://blueprints.launchpad.net/sahara/+spec/mount-share-api

As OpenStack's shared file provisioning service, manila offers great
integration potential with sahara, both for shared binary storage and as a
data source. While it seems unnecessary to wrap manila's share provisioning
APIs in sahara, allowing users to easily mount shares to all nodes of a Sahara
cluster in a predictable way will be a critical convenience feature for this
integration.

Problem description
===================

Manually mounting shares to every node in a large cluster would be a tedious
and error-prone process. Auto-mounting shares that are requested for use in
either the data source or binary storage case might be feasible for some use
cases. However, outside of our (optional) EDP interface this functionality
would never be usable. As such, it is best to provide the user an API for
mounting of shares onto Sahara clusters.

Proposed change
===============

This change proposes to expand the node group template, node group, cluster
template, and cluster API resources and database objects to contain a "shares"
field. As per other node group fields, the cluster template and cluster APIs
will allow overwrite of this field (which is particularly critical for
composition, given that manila shares represent concrete data rather than
abstract resource pools.) At the resource level (for both resource types),
this field will be defined by the following jsonschema:

::

    "shares": {
        "type": "array",
        "items": {
            "type": "object",
            "properties": {
                "id": {
                    "type": "string",
                    "format": "uuid"
                },
                "path": {
                    "type": ["string", "null"]
                },
                "access_level": {
                    "type": ["string", "null"],
                    "enum": ["rw", "ro"],
                    "default": "rw"
                }
            },
            "additionalProperties": False,
            "required": [
                "id"
            ]
        }
    }

"id" above refers to the UUID of the manila share to be mounted. It is
required.

"path" refers to the local path on each cluster node on which to mount this
share, which should be universal across all nodes of the cluster for
simplicity. It will default to ``/mnt/{share_id}``.

"access_level" governs permissions set in manila for the cluster ips.
This defaults to 'rw'.

Because no part of this field requires indexing, it is proposed that the
above structure be directly serialized to the database as a TEXT field in
JSON format.

Any share specified at the node group level will be mounted to all instances
of that node group. Any share specified at the cluster level will be mounted
to all nodes of that cluster. At cluster creation, in the case that a specific
share id is specified at both the node group and cluster level, the cluster's
share configuration (path and access level) will entirely replace the node
group's configuration. Any merge of share configurations is seen as needlessly
complex and error-prone, and it is our longstanding pattern that cluster-level
configurations trump node group-level configurations.

Error cases in this API include:

1. The provided id is not a valid manila share id, as assessed via
   manilaclient with the user's credentials.
2. The provided path is not a valid, absolute Linux path.
3. Path is not unique (within the set of shares specified for any one node
   or the set of shares specified for any cluster.)
4. The provided id maps to a manila share type which sahara is not currently
   equipped to mount.
5. No manila service endpoint exists within the user's service catalog.

On cluster creation (or update, if update becomes an available endpoint,)
just after the cluster becomes available and before delegating to the plugin
(such that any shares intended for HDFS integration will be in place for
the plugin configuration to act upon,) Sahara will execute a share mounting
step. For each share, Sahara will take the following steps:

1. Query manila for share information, including share type and defaults.
2. Query the cluster object to find internal ip addresses for all cluster
   nodes of any node group for which the share should be mounted.
3. For each such node, call to manila to allow access for each ip according
   to the permissions set on the share.
4. Make a remote call to each qualifying node and mount the share via its
   mount address as returned from manila.

Steps 1-3 above will be handled via common code in an abstract ShareHandler
class. The last step will be delegated to a concrete instance of this class,
based on share type as reported by manila, which will execute appropriate
command-line operations over a remote socket to mount the share.

The reference and test implementation for the first revision of this feature
will only provide an NFS mounter. An HDFS mounter is the next logical step,
but this feature set is already being worked on in parallel to this change and
falls outside of the scope of this specification.

Unmounting is a natural extension of this class, but is not covered in this
specification.

Alternatives
------------

A more-seamless approach to manila share storage and data sourcing could be
attempted, in which no API is exposed to the user, and shares are automatically
mounted and unmounted when resources on the share in question are needed (as
referenced in a data source URL or binary storage path). However, giving the
user the ability to mount and unmount shares at will may allow use cases which
we do not anticipate, and particularly in the context of usage of a sahara-
provisioned cluster without use of the EDP API, the new API is critical.

It would also be possible to attempt to wrap manila share creation (or even
share network creation or network router configuration) in sahara. It seems
reasonable, however, to assert that this would be an overstep of our charter,
and that asking users to create shares directly through manila will allow them
much fuller and up-to-date access to manila's feature set.

On the sahara implementation side, it would be possible to create a new
'share' resource and table, for ease of update and compositional modelling.
However, shares will likely never be a top-level noun in sahara; it seems that
a field is a better fit for the degree of share management we intend to
undertake than an entire resource.

It should be noted that this specification does not attempt to deal with the
question of filesystem driver installation across n distributions of Linux and
m filesystem types; such an effort is better suited to many specifications and
change sets than one. For the first stage of this effort, NFS will be used as
the test reference filesystem type.

Note that both binary storage and data source integration are intentionally
not handled here. A binary storage specification will build on this spec, but
this spec is being posted independently such that the engineers working on data
source integration can propose revisions to only the changes relevant to their
needs.

Data model impact
-----------------

A new 'shares' TEXT field will be added to both node groups and node group
templates.

REST API impact
---------------

A new 'shares' field will be added to the resource for both node groups and
node group templates. This field will only allow create functionality in the
initial change, as cluster update is currently a sticking point in our API.

Other end user impact
---------------------

Python-saharaclient will need to be made aware of the new shares field on all
supported resources.

Deployer impact
---------------

None.

Developer impact
----------------

None.

Sahara-image-elements impact
----------------------------

None for the initial change; addition of specialized fs drivers in the
future may require image changes.

Sahara-dashboard / Horizon impact
---------------------------------

The share mounting feature in Horizon will likely require a separate tab on
all affected resources, and is left for a separate spec.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  egafford

Secondary assignee/reviewer:
  croberts

Work Items
----------

* API Resource modification and call validation.
* DB model modification and testing.
* Manila client integration with Sahara.
* Logical glue code on cluster provisioning.
* ShareMounter abstraction and NFS impl.
* Unit testing.
* Integration testing as feasible (will require manila in CI env for full CI.)
* Update of API WADL site.
* Horizon changes (in separate spec).
* Documentation.

Dependencies
============

This feature introduces a new dependency on python-manilaclient.

Testing
=======

Unit testing is assumed; beyond this, full integration testing will depend on
the feasibility of adding a manila endpoint to our CI environment. If this is
feasible, then our testing path becomes clear; if it is not, then gated
integration testing will not be possible.

Documentation Impact
====================

This feature will require documentation in features.rst, and will drive changes
to the api documentation.

References
==========

See https://wiki.openstack.org/wiki/Manila/API if unfamiliar with manila
operations.
