..
  Technote content.

  See https://developer.lsst.io/docs/rst_styleguide.html
  for a guide to reStructuredText writing.

  Do not put the title, authors or other metadata in this document;
  those are automatically added.

  Use the following syntax for sections:

  Sections
  ========

  and

  Subsections
  -----------

  and

  Subsubsections
  ^^^^^^^^^^^^^^

  To add images, add the image file (png, svg or jpeg preferred) to the
  _static/ directory. The reST syntax for adding the image is

  .. figure:: /_static/filename.ext
     :name: fig-label
     :target: http://target.link/url

     Caption text.

   Run: ``make html`` and ``open _build/html/index.html`` to preview your work.
   See the README at https://github.com/lsst-sqre/lsst-technote-bootstrap or
   this repo's README for more info.

   Feel free to delete this instructional comment.

:tocdepth: 1

.. Please do not modify tocdepth; will be fixed when a new Sphinx theme is shipped.

.. sectnum::

.. Add content below. Do not include the document title.

.. note::

   **This technote is not yet published.**

   SQuaRE instructions for making official releases



The LSST Stack Release Process
------------------------------

The LSST stack is a large mixed Python/C++ codebase split among many repos with
multiple dependencies between them built by a niche build system. The release
process is therefore a bit complex and can be protracted. In an attempt not to
have to co-ordinate the state of the release with 80+ developers, or worse,
have them down tools while a release is being minted, this process has been
optimized to impact development as little as possible. Unless a developer needs
to be called upon to fix a failing integration or platform test, she can carry
on as normal.

In gross, the conceptual steps are:

#. A successful weekly release is identified that forms the basis for the
   official release.

#. The first release candidate (``rc1``) is tagged using that weekly as a seed.
   This means that it does not matter if the codebase's master has moved on in
   the meanwhile.

#. Repositories are tagged or branched with additional rc tags if hotfixes or
   late features are accepted

#. Documentation is updated on a branch (release notes, installation
   instructions, metrics report)

#. The final build is published and the documentation merged to master.



Kicking-off the process
-----------------------

The first step is to establish whether it is a good time to make a release, eg.
if many developers are about to push significant features, it may be better to
wait for them to finish. Announce the intent to make a release in the #dm
channel and inform all DM T/CAMs and the DM System Engineer.

If it is generally a good time, identify the nearest weekly release, this will
be the seed for the release candidate.

Next, start a community.lsst.org post to use for status updates.



Preparing for a candidate release
---------------------------------

Get codekit installed
^^^^^^^^^^^^^^^^^^^^^

The release manager can folow the instructions at:

https://github.com/lsst-sqre/sqre-codekit

If you are working interactively you want to set the environment variable
``DM_SQUARE_DEBUG`` to ``1`` as it gives more insight on what is going on with
the Github REST API calls.


Github teams
^^^^^^^^^^^^

There are three "special" teams in the LSST Github org:

- ``Data Management``

- ``DM Externals``

- ``DM Auxilliaries``

These are used in the release process in the following way:

- ``Data Management`` repos are a dependency of ``lsst_distrib`` and should be
  tagged with the bare release version, eg. ``14.0``, unless the repo is also a
  member of the ``DM Externals`` team.  All repos tagged as part of a release
  should be members of the ``Data Management`` team to ensure that DM
  developers are able to modify all components of a release.

- ``DM Externals`` also indicates a dependency of ``lsst_distrib`` but one that
  is tagged with a ``v`` prefix in front of the release version. Eg., ``v14.0``
  This is required because ``lsst-build`` derives the eups product version
  string from git tags that begin with a number.  DM developers prefer that
  eups display external packages version string rather than of a DM composite
  release. Thus the ``v`` prefix causes the git tag to be ignored by
  ``lsst_distrib``.  "External" repos must not also be members of ``DM
  Auxilliaries``.

- ``DM Auxilliaries`` are repos that we want to snapshot as part of a release
  but are not an eups dependency of ``lsst_distrib``. "Aux" repos must not also
  be members of ``DM Externals``.

Branching the docs
^^^^^^^^^^^^^^^^^^

At this point you should branch ``lsst/pipelines_lsst_io``

.. code-block:: bash

   git clone https://github.com/lsst/pipelines_lsst_io.git
   git checkout -b 14.0


Candidate release
-----------------

Identify the weekly you wish to build the release on, say ``w_2017_33``.

Look into the text of the annotated tag (using git log or the Github UI) to see
what the ``bXXXX`` number was.


Tag the candidate
^^^^^^^^^^^^^^^^^

Tagging first and third parties allows the release to be reproduceable in the
future and is necessary for the final build process.

Note the difference in tag name convention between first and third parties.

.. code-block:: bash

   # git tag the first parties

   github-tag-version --org lsst --team 'Data Management' --candidate v14_0_rc2 14.0 bXXXX
   github-tag-version --org lsst --team 'Database' --candidate v14_0_rc2 14.0 bXXXX

   # for externals NOTE THE v PREFIX to avoid stomping on the eups semantic versioning
   github-tag-version --org lsst --team 'DM Externals' --candidate v14_0_rc2 v14.0 bXXXX

This is the final tag against the third parties since they are slow-moving and
have been proven to work with the weekly candidate seed. In the rare event
where a problem is identified the tag can be moved along.


Publish the candidate
^^^^^^^^^^^^^^^^^^^^^



Final source release
--------------------


Branching lsst
^^^^^^^^^^^^^^^

In this process we make use of the fact that git doesn't care whether a ref is
a tag or a branch to constrain the number of branches to repositories that need
retroactive maintainance or need to be available in more than one cadence. One
such example is the ``lsst`` repo since it containes ``newinstall.sh`` which
sets the version of eups, and that may be different for an official release
than the current bleed.

The first repo that should be branched is lsst/lsst:

.. code-block:: bash

   git clone https://github.com/lsst/lsst.git
   git checkout -b 14.0

Now in ``lsst/scripts/newinstall.sh`` change the canonical reference for this
newinstall to be one associated with the current branch::

  NEWINSTALL="https://raw.githubusercontent.com/lsst/lsst/14.0/scripts/newinstall.sh"

and commit and push.

This means that if you need to update newinstall.sh for bleed users,
official-release users will not be prompted to update to the latest version,
but will phone home against their official-release branch for hotfixes.

Also double-check for other things that might need to be updated, like the
documentation links (though these should really be fixed on master prior to
branching or cherry-picked back).

Doc update: newinstall.rst
^^^^^^^^^^^^^^^^^^^^^^^^^^

Update the ``newinstall.rst`` page on your release branch of pipelines_lsst_io
with the new download location of the newinstall.sh script.


Final tag
^^^^^^^^^

Now it's time to lay down the final git tag. For repositories that have already
been branched with the ``14.0`` ref, that will fail, which is fine.

This is mostly a repeat of the process for laying down the candidate tag but
this time we use numeric tags so that eups will see them::

  # tag repos involved in the final candidate and final build
  github-tag-version --org lsst --team 'Data Management' --candidate v14_0_rc2 14.0 b3176
  github-tag-version --org lsst --team 'Database' --candidate v14_0_rc2 14.0 b3176

Since you already tagged the third parties with their special final tag
already, no need to do anything here.

Release build
^^^^^^^^^^^^^

- Submit the run-rebuild job with your parameters (eg. ``14.0`` ``v14.0``)

- At this point you should not be seeing master-g type references as eups
  versions. Everything should have a tag-derviced version such as ``14.0`` if
  they are a DM repo and their semantic tag (eg. ``pyfits 3.0``) if they are
  external.  If you see one, you need to chase down why. The only situation
  that should happen is if a third party but a branch is used for LSST
  development that lacks any other type of semantic versioning (in the ``14.0``
  release this included starlink_ast and jointcal_cholmod.

- Note your final ``bNNNN`` number for the publish (either from the build log
  or by looking at the next of the annotated ``14.0`` tag on any repo eg. afw).

- Submit the run-publish job making sure you have selected ``package`` and not
  ``git`` as the option.


Other OS checking
^^^^^^^^^^^^^^^^^

While we only officially support the software on certain platforms
(`RHEL/CentOS 7` is the reference, and we CI `MacOS` and `RHEL 6`), we check in
a number of other popular platforms (eg `Ubuntu`, newer versions of `CentOS`
etc) by spinning up machines on Digital Ocean (typically) and following the
user install instructions. This also allows us to check the user from-scratch
installation instructions including the pre-requisites.



Binaries
--------

Run the tarball-matrix job with the options ``SMOKE``, ``RUN_SCONS_CHECK``,
``PUBLISH``.


Documentation
-------------

Documentation to be collected for the release notes in ``pipelines_lsst_io``
is:

- Release notes from the T/CAMs for Pipelines, SUI, and DAX
- Characterisation report from the DM or SQuaRE scientist
- Known issues and pre-requisites from the T/CAM for SQuaRE
- Before merging to master, ask the Documentation Engineer to review


c.l.o stubb
-----------

.. code-block:: none

  Here is where we currently are in the release process. Current step in bold.

  Summary
  -----------

  Release is complete

  Precursor Steps
  ---------------------------------

  1. Identify any pre-release blockers ("must-have features") :tools:
  2. Wait for them to clear


  Release Engineering Steps
  -------------------------------

  1. Eups publish rc1 candidate (based on b2748) (also w_2017_33)
  1. Git Tag v14.0-rc1
  1. Branch v14 of newinstall.sh
  1. **Wait for first round of bugs to clear**
  1.Repeat last 2 steps, -rcN candidates  <-- final candidate is rc1 [yay!]
  1. Confirm DM Externals are at stable tags
  1. Tag DM Auxilliary (non-lsst_distrib) repos
  1. Full OS testing (see https://ls.st/faq )
  1. Git Tag 14.0, rebuild, eups publish

  Binary release steps
  ------------------------

  1. Produce factory binaries
  1. Test factory binaries
  1. Gather contributed binaries

  Documentation Steps
  -------------------------

  1. Update Prereqs/Install
  1. Update Known Issues
  1. Gather Release notes
  1. Gather Metrics report
  1. **Email announcement**



Format of "tags"
----------------

git tags
^^^^^^^^

- DM produced code this is part of an "official" release  **must** have a git
  tag that starts with a *number*

- "official" release git tags on external/third-party software that DM has
  repackaged must be prefixed with a ``v`` but are otherwise identical to that
  on DM produced code. Eg., ``42.0.0 -> v42.0.0``

- Non-"official" releases, release candidates, weekly builds, etc. **must**
  start with a *letter*

- **shall** only use ``[a-z]``, ``[0-9]``, and ``.``

  * *lowercase* latin alphabet characters **shall** be used; *uppercase*
    characters are forbidden

  * These common characters **must not** be used: ``-``, ``_``, ``/``


Examples of *valid* (good) git tags

.. code-block:: none

  # unofficial builds
  d.2038.01.19
  w.2038.03

  # release candidate
  v42.0.0.rc99

  # official release of DM produced code
  42.0.0

  # official release of external/third-party product
  v42.0.0

Examples of *invalid* (bad) git tags

.. code-block:: none

  d_2038_01_19
  w_2038_03
  v42-0-0-rc99
  42_0_0
  v42_0_0
  foo/bar

eups tags
^^^^^^^^^

- **must not** start with a numeric value

- **shall** only use ``[a-z]``, ``[0-9]``, and ``_``

  * *lowercase* latin alphabet characters **shall** be used; *uppercase*
    characters are forbidden

  * EUPS reportedly has or has had problems with ``.`` and ``-``

- official releases and release candidates **must** be prefixed with ``v``


Examples of *valid* (good) eups tags

.. code-block:: none

  # unofficial builds
  d_2038_01_19
  w_2038_03

  # release candidate
  v42_0_0_rc99

  # official release of DM produced code AND external/third-party product
  v42_0_0

Examples of *invalid* (bad) eup tags

.. code-block:: none

  123
  d.2038.01.19
  w.2038.03
  v42_0_0-rc99
  42.0.0
  v42.0.0
  foo/bar

git <-> eups tag conversion
^^^^^^^^^^^^^^^^^^^^^^^^^^^

The "tags" along each row in the following table should be considered
equivalent conversions.

============  ============  ========
internal git  external git  eups tag
============  ============  ========
d.2038.01.19  d.2038.01.19  d_2038_01_19
w.2038.03     w.2038.03     w_2038_03
v42.0.0.rc99  v42.0.0.rc99  v42_0_0_rc99
42.0.0        v42.0.0       v42_0_0
============  ============  ========

Conda Environment/Packages Update
---------------------------------

There are conflicting pressures of updating the conda package list frequently
to minimize the ammount of [likely] breakage at one time and resisting changes
as the git ``sha1`` of the conda environment files is used to defined the
``ABI`` of the eups ``tarball`` packages.

Adding a new Conda package
^^^^^^^^^^^^^^^^^^^^^^^^^^

#. The name of the package needs to "bleed" or un-versioned environment files in
   the ``lsst/lsstsw`` repo. Which are:

    - https://github.com/lsst/lsstsw/blob/master/etc/conda3_bleed-linux-64.txt
    - https://github.com/lsst/lsstsw/blob/master/etc/conda3_bleed-osx-64.txt

    These env files are currently kept in the original conda environment file
    format and have not yet been migrated to the newer ``yaml`` based format as
    it only works with fairly recent conda releases. (*TODO* migrate to `yaml`
    format after DM-14011 is merged).

    The bleed env files should be keep in sync with the *exception* of the
    ``nomkl`` package, which is required on ``linux``.  Also note that the env
    files should be kept sorted to allow for clean ``diff`` s.

#. The regular conda env files need to be updated by running a fresh install
   with ``deploy -b``` (bleed install) and then manually exporting the env to a
   file.  A side effect of this is other package versions will almost certainly
   change and this **is an ABI breaking event**. The existing env files are:

    - https://github.com/lsst/lsstsw/blob/master/etc/conda3_packages-linux-64.txt
    - https://github.com/lsst/lsstsw/blob/master/etc/conda3_packages-osx-64.txt

    ``conda list -e`` should be run on ``linux`` and ``osx`` installs and the
    results committed for both platforms as **a single commit** so that the the
    abbrev sha1 of the latest commit for both files will be the same.

#. As an abbreviated sha1 of the ``lsst/lsstsw`` repo is used to select which
   [version of] conda env files are used and to define the eups binary tarball
   "ABI", jenkins needs to know this value to ensure that ``newinstall.sh`` is
   explicitly using the correct ref and to construct the paths of the tarball
   ``EUPS_PKGROOT`` s.  The ``lsstsw_ref`` / ``LSST_LSSTSW_REF`` needs to be
   updated at:

    - https://github.com/lsst-sqre/jenkins-dm-jobs/blob/master/etc/scipipe/build_matrix.yaml#L10
    - https://github.com/lsst/lsst/blob/master/scripts/newinstall.sh#L33

#. The ~last major release should be rebuilt in the new "ABI" ``EUPS_PKGROOT`` so
   that that newinstall.sh from master will still be able to do a binary
   install of the current major release.  This may be done by triggering a
   jenknins ``release/tarball-matrix`` build.
