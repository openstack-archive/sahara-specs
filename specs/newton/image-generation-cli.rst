..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=========================================
 CLI for Plugin-Declared Image Generation
=========================================

This specification describes the next stage of the image generation epic,
and proposes a CLI to generate images from the plugin-defined recipes
described in the validate-image-spi_ blueprint.

Problem description
===================

https://blueprints.launchpad.net/sahara/+spec/image-generation-cli

Sahara has historically used a single monolithic repository of DIB
elements for generation of images. This poses several problems:

1) Unlike most use cases for DIB, Sahara end-users are not necessarily
   OpenStack experts, and are using OpenStack as a means to manage
   their data processing clusters, but desire to customize their nodes
   at the image level. As these users are concerned with data processing
   over cloud computing, ease of image modification is critical in the
   early stages of technology evaluation. The Sahara team has found that
   while DIB is a very powerful tool, it is not overly friendly to new
   adopters.
2) In order to mature to a stage at which new Sahara users can add
   features to the codebase cleanly, Sahara must draw firmer lines
   of encapsulation around its plugins. Storing all image generation
   recipes (whatever the implementation) in a single repository that
   cuts across all plugins is counter to the end goal of allowing
   the whole functionality of a new plugin to be exposed at a single
   path.
3) Sahara end users will very seldom need to modify base OS images in
   a particularly deep way, and will seldom need to create new images.
   The changes Sahara must make to images in order to prepare them for
   use are actually quite basic: we install packages and modify
   certain configuration files. Sacrificing some power and speed for
   some ease of use is very reasonable in our use case.

For these reasons, a means to encapsulate image generation logic within
a plugin, a command line utility to exercise these recipes, and a
backing toolchain that emphasizes reliability over flexibility and
speed are indicated.

Proposed change
===============

The https://blueprints.launchpad.net/sahara/+spec/validate-image-spi
blueprint describes the first steps of this epic, enabling us to
define recipes for image validation and generation of clusters from
"clean" images.

CLI input
---------

The current specification proposes that a CLI script be added to
sahara at sahara.cli.sahara_pack_image.py. This script will not
start a new service as others do; rather, it will use the pre-existent
sahara.services.images module to run the image generation recipe on a
base image through a new remote implementation. This remote
implementation will use libguestfs'python API to alter the image to the
plugin's specifications.

--help text for this script follows:

::

    Usage: sahara-image-create
        --image IMAGE_PATH
        [--root-filesystem]
        [--test]
        PLUGIN PLUGIN_VERSION
        [More arguments per plugin and version]
    * --image: The path to an image to modify. This image will be modified
        in-place: be sure to target a copy if you wish to maintain a
        clean master image.
    * --root-filesystem: The filesystem to mount as the root volume on the
        image. No value is required if only one filesystem is detected.
    * --test-only: If this flag is set, no changes will be made to the
        image; instead, the script will fail if discrepancies are found
        between the image and the intended state.
    [* variable per plugin and version: Other arguments as specified in
      the generation script.
    * ...]
    * -h, --help: See this message.


Both PLUGIN and PLUGIN_VERSION will be implemented as required subcommands.

If --test-only is set, the image will be packed with the ``reconcile``
option set to False (meaning that the image will only be tested, not changed.)

Each image generation .yaml (as originally described in the
validate-images-spi spec,) may now register a set of 'arguments' as well
as a set of 'validators'. This argument specification should precede the
validator declaration, and will take the following form:

::

    arguments:
      - java-version:
        description: The java distribution.
        target_variable: JAVA_VERSION
        default: openjdk
        required: false
        choices:
          - oracle-java
          - openjdk

A set of arguments for any one yaml must obey the following rules:

1) All argnames must be unique, and no argnames may collide with global script
   argument tokens.
2) If required is false, a default value must be provided.
3) The target_variable field is required, for clarity.

An ImageArgument class will be added to the sahara.service.images module in
order that all plugins can use the same object format. It will follow the
object model above.

New SPI methods
---------------

In order to facilitate the retrieval of image generation arguments, a
get_image_arguments SPI method will be added to the plugin SPI. The arguments
returned from this method will be used to build help text specific to a plugin
and version, and will also be used to validate input to the CLI. It will
return a list of ImageArguments.

A pack_image method will also be added to the plugin SPI. This method will
have the signature:

::

    def pack_image(self, hadoop_version, remote, reconcile=True,
                   image_arguments=None)

This method will take an ImageRemote (see below). In most plugins, this
will:

1) Validate the incoming argument map (to ensure that all required arguments
   have been provided, that values exist in any given enum, etc.)
2) Place the incoming arguments into an env_map per their target_variables.
3) Generate a set of ImageValidators (just as is done for image validation).
4) Call the validate method of the validator set using the remote and
   argument_map.

However, the implementation is intentionally vague, to allow plugins to
introduce their own image packing tools if desired (as per the
validate-images-spi spec.)

ArgumentCaseValidator
---------------------

Now that these image definitions need to be able to take arguments, a new
ArgumentCaseValidator will be added to the set of concrete
SaharaImageValidators to assist image packing developers in writing clean,
readable recipes. This validator's yaml definition will take the form:

::

    argument_case:
        argument_name: JAVA_VERSION
        cases:
            openjdk:
                - [action]
            oracle-java:
                - [action]

The first value case key which matches the value of one of the variable will
execute its nested actions. All subsequent cases will be skipped.

ImageRemote
-----------

A new ImageRemote class will be added to a new module, at
sahara.utils.image_remote. This class will be encapsulated in its own module
to allow distribution packagers the option of externalizing the dependency on
libguestfs into a subpackage of sahara, rather than requiring libguestfs as a
dependency of the main sahara python library package in all cases.

This class will represent an implementation of the sahara.utils.remote.Remote
abstraction. Rather than executing ssh commands from the provided arguments,
however, this class will execute scripts on a target image file using
libguestfs' python API.

The CLI will use generate a remote targeting the image at the path specified
by the 'image' argument, and use it to run the scripts which would normally be
run over ssh (on clean image generation) within the image file specified.

Alternatives
------------

We have discussed the option of bringing DIB elements into the Sahara plugins.
However, this was nixed due to the issues above related to tradeoff between
speed and power and usability, and because of certain testing issues (discussed
in an abandoned spec in the Mitaka cycle).

It is also possible that we could maintain our current CLI in
sahara-image-elements indefinitely. However, as more plugins are developed,
a single monolithic repository will become unwieldy.

Data model impact
-----------------

None.

REST API impact
---------------

None.

Other end user impact
---------------------

The new CLI will be (at present) the only means of interacting with this
feature.

Deployer impact
---------------

Packaging this script as a separate python (and Debian/RPM package) is an
option worth discussing, but at this time it is intended that this tool
be packed with the Sahara core.

Developer impact
----------------

This feature will reuse definitions specified in the validate-image-spi_ spec,
and thus should have minimal developer impact from that spec.

Sahara-image-elements impact
----------------------------

This feature will hopefully, once it reaches full maturity and testability,
supplant sahara-image-elements, and will provide the baseline for building
image packing facility into the sahara API itself.

Sahara-dashboard / Horizon impact
---------------------------------

No dashboard representation is intended at this time.


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  egafford

Other contributors:
  None

Work Items
----------

1) Create and unit test images.py changes.
2) Test plugin-specific image packing for any plugins to implement this
   feature. Intended first round of plugins to implement:
3) Implement CI tests for this feature.

Dependencies
============

This feature will introduce a dependency on libguestfs, though it will only
use libguestfs features present in all major distributions.


Testing
=======

As this feature does not touch the API, it will not introduce new Tempest
tests. However, testing the image generation process itself will require
attention.

It is proposed that tests be added for image generation as each plugin
implements this generation strategy, and that nightly tests be created to
generate images and run these images through cluster creation and EDP testing.

These should be implemented as separate tests in order to quickly
differentiate image packing failure and cluster failure.

As these recipes stabilize for any given plugin, we should begin to run
these tests when any change to the sahara repository touches image
generation resources for a specific plugin (which should be well-encapsulated
in a single directory under each version for each plugin.) Toward the end of
this epic (as we are nearing the stage of authoring the API to pack images,
we may consider removing integration tests for SIE to save lab time. Still,
these tests will be, compared to sahara service tests, very resource-light
to run.


Documentation Impact
====================

This feature should be documented in both devref (for building image
generation recipes) and in userdoc (for script usage).

References
==========

None.

.. _validate-image-spi: https://blueprints.launchpad.net/sahara/+spec/validate-image-spi
