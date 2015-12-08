..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

============================
SPI Method to Validate Image
============================

https://blueprints.launchpad.net/sahara/+spec/validate-image-spi

This specification details the addition of a method to the Sahara plugin SPI
to validate that a chosen image is up to the specification that the plugin
requires. While it is not expected that plugin writers will be able to test
the image deeply enough to ensure that a given arbitrary image will succeed
in cluster generation and be functional in all contexts, it is hoped that by
implementing this method well, plugin authors can provide a well-defined,
machine-actionable contract which will be versioned with the plugin itself.

Problem description
===================

At present, Sahara's image generation and cluster provisioning features are
almost entirely decoupled: sahara-image-elements generates an image, and this
image is taken by the server and assumed to be valid. This introduces the
possibility of version incompatibility between sahara-image-elements and
sahara itself, and failure (complete or partial, immediate or silent) in the
case of the addition or modification of features on either side.

Larger context
--------------

This issue is only one part of a larger problem, which will not be wholly
addressed in this spec, but for which this spec is an incremental step toward
a solution.

At present, the processes involved in image generation and use are:

1) Image packing (pre-cluster spawn)
2) Clean image provisioning (building a cluster from an OS-only base image)
3) Image validation (ensuring that a previously packed image really is ok to
   use for the plugin, at least for the rules we can easily check)

The first, image-packing, is currently only possible via a command line
script. The ideal user experience would allow generation of images either
outside of OpenStack, via a command-line script, or with OpenStack, via a
sahara API method. At present, this is not possible.

The second, in our present architecture, requires essentially rewriting the
logic required to generate an image via the command line process in the plugin
code, leading to duplicate logic and multiple maintenance points wherever
cluster provisioning from clean images is allowed. However, it should be noted
that in the clean image generation case, this logic is in its right place
from an encapsulation perspective (it is maintained and versioned with the
plugin code, allowing for easy separation, rather than maintained in a
monolithic cross-cutting library which serves all plugins.)

The third is not formally undertaken as a separate step at all; it will be
implemented by the feature this specification describes.

Within the context of this larger problem, this feature can be seen as the
first incremental step toward a unified solution for image validation,
unification of clean and packed image generation logic, and facilitation of
image packing via an API. Once this SPI method is stable, functional, and
expresses a complete set of tests for all maintained plugins, the validation
specification can then be reused as a series of idempotent state descriptions
for image packing, which can then be exposed via an API for any plugins which
support it.

Proposed change
===============

SPI method contract
-------------------

A new method will be added to the plugin SPI in Sahara:

::

    validate_images(self, cluster, reconcile=True)

This method will be called after cluster provisioning (as this will be
necessary for machine access) and before cluster configuration. This method
will receive the cluster definition as an argument, as well as a boolean flag
describing whether or not the plugin should attempt to fix problems if it
finds them.

If this method is not implemented by a plugin, provisioning will proceed as
normal; as this is purely a safety feature, full backward compatibility with
previous plugin versions is acceptable.

The contract of this method is that on being called, the plugin will take
any steps it sees fit to validate that any utilized images are fit for their
purposes. It is expected that all tests that are run will be necessary for
the cluster to succeed, but not that the whole set of tests will be
absolutely sufficient for the cluster to succeed (as this would essentially
be disproving a universal negative, and would require such in-depth testing as
to become ludicrous.)

If the reconcile flag is set to False, this instructs the plugin that it
should only test the image, but change nothing, and report error if its tests
fail. If reconcile is True (this will be set by default,) then the plugin will
also take any steps it is prepared to take to bring the instances of the
cluster into line with its expectations. Plugins are not required to provide
this functionality, just as they are not required to implement validate_image;
if they wish to fail immediately in the case of an imperfect image, that is
their choice. However, if a plugin does not support reconciliation, and
reconcile is set to True, it must raise an error; likewise, if a plugin
receives reconcile=False but it is not able to avoid reconciliation (if, for
instance, its implementation uses Puppet and will by definition make changes
if needed,) it must raise as well.

sahara.plugins.images
---------------------

The sahara base service will provide a set of utilities to help plugin authors
to validate their images. These will be found in sahara.plugins.images. Usage
of these utilities is wholly optional; plugin authors may implement validation
using whatever framework they see fit. It is noted that this module could be
immediately written to allow a great deal of deep functionality in terms of
matching image validations to services, allowing custom images to be used for
specific nodegroups and service sets. However, as no plugins are currently
implementing such a feature set, a more basic first iteration is reasonable,
and the methods described below will allow a plugin author to perform such
specific validations if it is desired.

The images module will provide several public members: the definitions of
the most notable (if not all) are given below:

::

    def validate_instance(instance, validators, reconcile=True, **kwargs):
        """Runs all validators against the specified instance."""

    class ImageValidator(object):
    """Validates the image spawned to an instance via a set of rules."""

        __metaclass__ = abc.ABCMeta

        @abc.abstractmethod
        def validate(self, remote, reconcile=True, **kwargs):
            pass

    class SaharaImageValidatorBase(ImageValidator):
        """Still-abstract base class for Sahara's native image validation,
        which provides instantiation of subclasses from a yaml file."""

        @classmethod
        def from_yaml(cls, yaml_path, validator_map, resource_roots):
            """Constructs and returns a validator from the provided yaml file.
            :param yaml_path: The path to a yaml file.
            :param validator_map: A map of validator name to class. Each class
                is expected to descend from SaharaImageValidator. This method
                will use the static map of validator name to class provided in
                the sahara.plugins.images module, updated with this map, to
                parse he appropriate classes to be used.
            :param resource_root: The roots from which relative paths to
                resources (scripts and such) will be referenced. Any resource
                will be pulled from the first path in the list at which a file
                exists."""

    class SaharaImageValidator(SaharaImageValidatorBase):
        """The root of any tree of SaharaImageValidators."""

        def validate(self, remote, reconcile=True, env_map=None, **kwargs):
            """Validates the image spawned to an instance."""
            :param env_map: A map of environment variables to be passed to
                scripts in this validation."""

Additionally, two classes of error will be added to sahara.plugins.exceptions:

* ImageValidationError: Exception indicating that an image has failed
  validation.
* ImageValidationSpecificationError: Exception indicating that an image
  validation spec (yaml) is in error.

SaharaImageValidator
--------------------

It is entirely possible for a plugin author, in this framework, to use
idempotent state enforcement toolsets, such as Ansible, Puppet, Chef, and the
like, to validate and reconcile images. However, in order that Sahara need
not absolutely depend on these tools, we will provide the
SaharaImageValidator class.

This validator will provide a classmethod which allows it to build its
validations from a .yaml file. The first iteration of this validator will be
very limited, and as such will provide only a few abstract validation types.
This yaml will be interpreted using whatever ordering is available; as dicts
are unordered in yaml, this scheme makes extensive use of lists of single-
item dicts.

An example .yaml file showing the revision-one validator set follows. Note
that these are not intended to be realistic, sahara-ready definitions, merely
examples taken from our experience:

::

    validators:
      - os_case:
          - redhat:
              - package: nfs-utils
          - debian:
              - package: nfs-common
      - any:
        - package: java-1.8.0-openjdk-devel
        - package: java-1.7.0-openjdk-devel
      - script: java/setup-java-home
      - package:
        - hadoop
        - hadoop-libhdfs
        - hadoop-native
        - hadoop-pipes
        - hadoop-sbin
        - hadoop-lzo
        - lzo
        - lzo-devel
        - hadoop-lzo-native

These resource declarations will be used to instantiate the following basic
validator types:

Validator types
---------------

SaharaPackageValidator (key: package)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Verifies that the package or packages are installed. In the reconcile=True
case, ensures that local package managers are queried before resorting to
networked tools, along the lines of:

::

    `dpkg -s $package || apt-get -y install $package`  # debian
    `rpm -q $package || yum install -y $package`  # redhat

The input to this validator may be a single package definition or a list of
package definitions. If the packages are grouped in a list, any attempt to
install the packages will be made simultaneously. A package definition may be
a single string or a nested structure, which may support a version attribute
as follows:

::

    - package: hadoop
    - package:
      - hadoop-libhdfs
      - lzo:
          version: xxx.xxx

Because reliable version comparison will often require reference to epochs,
and because the tool must succeed in an offline context, the initial, Sahara
core-provided package validator will allow only exact version pinning. As this
version is yaml-editable, this is not adequate to our purposes, and can be
extended by plugin developers if needed and appropriate.

SaharaScriptValidator (key: script)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Runs an arbitrary script from source, as specified by a relative path from the
resource root.

The input to this validator must be a single script definition. A script
definition may be a single string or a nested structure, which may support
attributes as follows (the example is purely explanatory of the structure):

::

  - script: simple_script.sh
  - script:
    java/find_jre_home:
      output: JRE_HOME  # Places the stdout of this script into the env map
                        # for future scripts
  - script:
    java/setup_java_home:
      env_vars:         # Sets only the named env vars from the env map
        - JDK_HOME
        - JRE_HOME

Scripts are always provided the env var $SIV_DISTRO, which specifies the linux
distribution per our current SIE distro conventions, and the env var
$SIV_RECONCILE, which is set to 0 if only validation should occur and 1 if
corrective action should be taken.

Additional variables are referenced from the env_map argument passed
originally to SaharaImageValidator.from_yaml (and are presumably parsed from
cluster configuration information). The output attribute of the script
resource can be used to modify this map in flight, placing the output of a
script into the (single) named variable. More complex interactions require
extension.

This validator is intentionally lightweight. These image validations and
manipulations should not be overwhelmingly complex; if deep configuration is
needed, then the more freeform configuration engine should run those steps, or
the plugin author should utilize a more fully-featured state enforcement
engine, with all the dependencies that entails (or write a custom validator).

NOTE THAT ALL SCRIPTS REVIEWED BY THE SAHARA TEAM MUST BE WRITTEN TO BE
IDEMPOTENT. If they are to take non-reproducible action, they must test to see
if that action has already been taken. This is critical to the success of this
feature in the long term.

SaharaAnyValidator (key: any)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Verifies that at least one of the validators it contains succeeds. If
reconcile is true, runs all validators in reconcile=False mode before
attempting to enforce any. If all fail in reconcile=False mode, it then
attempts to enforce each in turn until one succeeds.

Note that significant damage can be done to an image in failed branches if
any is used with reconcile=true. However, guarding against this sort of
failure would impose a great deal of limitation on the use of this validator.
As such, warnings will be documented, but responsible use is left to the
author of the validation spec.

SaharaAllValidator (key: all)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Verifies that all of the validators it contains succeed. This class will be
instantiated by the yaml factory method noted above, and will contain all
sub-validations.

SaharaOSCaseValidator (key: os_case)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Switches on the distro of the instance being validated. Recognizes the OS
family names redhat and debian, as per DIB. Runs all validators under the
first case that matches.


Notes on validators
-------------------

Plugin authors may write their own validator types by extending the
SaharaImageValidator type, implementing the interface, and passing the key and
class into the validator_map argument of SaharaImageValidator.from_yaml.

It should be noted that current "clean" image generation scripts should be
moved into this layer as part of the initial effort to implement this method
for any given plugin, even if they are represented as a monolithic script
resource. Otherwise clean images will very likely fail validation.

Note also that the list above are certain to be needed, but as the implementer
works, it may become useful to create additional validators (file, directory,
and user spring to mind as possible candidates.) As such, the list above is
not necessarily complete; I hesitate, however, to list all possible validator
types I can conceive of for fear of driving over-engineering from the spec,
and believe that review of the design of further minor validator types can
wait for code review, so long as this overall structure is agreeable.

Alternatives
------------

We have many alternatives here.

First, to the problem of merging our validation, packing, and clean image
provisioning logic, we could opt to merge our current image generation code
with our service layer. However, this poses real difficulties in testing, as
our image generation layer, while functional, lacks the stability of our
service layer, and merging it as-is could slow forward progress on the project
as we wrestle with CI.

Assuming that we do not wish to merge our current image generation layer,
we could begin immediately to implement a new image generation layer in the
service side. However, this sort of truly revolutionary step frequently ends
in apathy, conflict, or both. Providing an image validation layer, with the
possibility of growing into a clean image generation API and, later, an image
packing API, is an incremental step which can provide real value in the short
term, and which is needed regardless.

Assuming that we are, in fact, building an image validation API, we could
wholly separate it from any image preparation logic (including clean image
provisioning.) There is a certain purist argument for separation of duties
here, but the practical argument that resource testing and enforcement are
frequently the same steps suggests that we should merge the two for
efficiency.

Assuming that we are allowing reconciliation of the image with the validation
layer, we could, instead of building our own lightweight validation layer,
demand that plugin authors immediately adopt one of Ansible, Puppet, Chef,
Salt, etc. However, three factors lead me not to embrace this option. First,
normal usage of these tools expects network access by default; in our
context, we do not want to use the external network unless absolutely
necessary, as our instances may not be network-enabled. While it is possible
to use them offline, it requires some care to do so, which might be offputting
for newcomers to Sahara who are versed in the chosen tool. Second, Sahara
should not be that opinionated about toolchains, either within our team or to
our userbase. Facilitating the usage of devops toolchains by providing a
clear, well-encapsulated API point is a good goal, but it is not Sahara's job
to pick a winner in that market. Third, such a framework is a significant
dependency for the sahara core, and such massive dependencies are always to be
regarded with suspicion. As such, providing a very lightweight framework for
validations is worthwhile, so that we do not need to depend absolutely on any
such framework, even in the short term before plugins are abstracted out of
the service repo.

Assuming that we do not wish to immediately adopt such a framework, we could
instead decide to immediately build a full-featured idempotent resource
description language, building many more validators with many more options.
While I may well have missed required, basic options, and welcome feedback, I
strongly suggest that we start with a minimal framework and build upon it,
instead of trying to build the moon from the outset. I have aimed in this spec
for extensibility over completeness (and as such have left some explicit
wiggle room in the set of validators to be implemented in the first pass.)

Data model impact
-----------------

None.

REST API impact
---------------

None; this change is SPI only.

Other end user impact
---------------------

For plugins using SaharaImageValidators, end-users will be able to modify the
.yaml files to add packages or run validation or modification scripts against
their images on spawn.

Deployer impact
---------------

None.

Developer impact
----------------

This SPI method is optional; plugins may, if they're feeling a bit cowboy
about things today, continue to spawn from any provided image without testing
it. As such, there is no strictly required developer impact with this spec.

Sahara-image-elements impact
----------------------------

None. Sahara-image-elements can keep doing its thing if this is adopted.
Future dependent specs may drive changes in how we expect images to be packed
(hopefully via an OpenStack API,) but this is not that spec, and can be
approved wholly independently.

Sahara-dashboard / Horizon impact
---------------------------------

None.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  egafford

Other contributors:
  ptoscano

Work Items
----------

* Add SPI method and call in provisioning flow; wrap to ensure that if absent,
  no error is raised.
* Build sahara.plugins.images as specified above, and all listed validators.
* Write .yaml files for CDH and Ambari plugins using this mechanism (other
  plugins may adopt over time, as the SPI method is optional.)
* Add unit tests.

Dependencies
============

No new dependencies (though this does provide an extension point for which
plugins may choose to adopt new dependencies.)

Testing
=======

Unit testing is assumed, as in all cases. The image validation mechanism
itself does not need extensive new integration testing; the positive case will
be covered by existing tests. Idempotence testing requires whitebox access to
the server, and is not possible in the scenario framework; if this system ever
is adopted for image generation, at that point we will have the blackbox hooks
to test idempotence by rerunning against a pre-packed image (which should
result in no change and a still-valid image.)

Documentation Impact
====================

We will need to document the SPI method, the SaharaImageValidator classes,
and the .yaml structure that describes them.

References
==========

None.
