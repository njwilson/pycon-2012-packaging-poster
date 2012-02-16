INTRODUCTION TO PACKAGING
=========================

TODO: Introduction to packaging in general


PAST/PRESENT
============

Python's Packaging Ecosystem
----------------------------

TODO: Overview of the current state of Python packaging


FUTURE
======

Highlights:

* Standard database of installed distributions (PEP 376)

  - Facilitates interoperability between package managers

  - Supports uninstallation

* Improved metadata (PEP 345)

  - Dependencies on Python versions (e.g., ``>=2.5,<2.7``)

  - Dependencies on distributions (i.e., name on PyPI) rather than modules

  - Static declaration of dependencies based on operating system, platform, and
    Python version/implementation

* Standard distribution versioning scheme (PEP 386)

  - Versions can be easily compared

  - Clear distinction between development, pre- and post-release, and final
    versions

* New packaging library: ``packaging``/``distutils2``

  - Obsoletes ``distutils``, ``setuptools``, and ``distribute``

  - Static configuration file (setup.cfg) replaces setup.py

  - Can be used as a library


New Standards
-------------

PEP 376 -- "Database of Installed Python Distributions"
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* Goal: Define a standard database of installed Python distributions so package
  managers are interoperable

* Motivation:

  - Existing tools (``distutils``,
    ``setuptools``/``distribute``/``easy_install``, and ``pip``) each have
    their own method of tracking installed distributions

  - Most of the existing tools don't support uninstallation

  - No common API exists for querying installed distributions

* This PEP defines a new standard, a ``.dist-info`` directory for each
  distribution with the following files:

  - ``METADATA`` - The distribution's metadata

  - ``RECORD`` - List of installed files

  - ``INSTALLER`` - Name of the installer that installed the project

  - ``REQUESTED`` - Indicates whether the project was installed explicitly or
    to satisfy a dependency

* Sample directory structure::

    site-packages/
    |
    |-- sample/
    |   |-- __init__.py
    |   |-- sample.py
    |
    |-- sample-1.0.0.dist-info/
        |-- INSTALLER
        |-- METADATA
        |-- RECORD
        |-- REQUESTED


PEP 345 -- "Metadata for Python Software Packages 1.2"
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* Goal: Define new metadata for distributions to better specify dependencies
  and relationships with other distributions

* Motivation:

  - Previous standards specified dependencies on Python *modules*, but people
    don't distribute *modules*; they distribute *distributions*

  - ``setuptools`` and ``distribute`` support dependencies on distributions
    (via ``install_requires``), but it's not standardized

  - Current methods require running code in ``setup.py`` to calculate
    dependencies based on operating system, platform, and Python
    version/implementation

* Primary new fields:

  +-------------------------+-------------------------------------------------------+----------------------------------------------+
  | Field Name              | Description                                           | Examples                                     |
  +=========================+=======================================================+==============================================+
  | | ``Requires-Python``   | Python version(s) that the distribution is guaranteed | | ``Requires-Python: 2.5``                   |
  |                         | to be compatible with                                 | | ``Requires-Python: >2.1``                  |
  |                         |                                                       | | ``Requires-Python: >=2.3.4``               |
  |                         |                                                       | | ``Requires-Python: >=2.5,<2.7``            |
  +-------------------------+-------------------------------------------------------+----------------------------------------------+
  | | ``Requires-External`` | External system dependencies (arbitrary string used   | | ``Requires-External: C``                   |
  |                         | as a hint to downstream project maintainers)          | | ``Requires-External: libpng (>=1.5)``      |
  +-------------------------+-------------------------------------------------------+----------------------------------------------+
  | | ``Requires-Dist``     | Names of Python distributions required, provided, and | | ``Requires-Dist: pkginfo``                 |
  | | ``Provides-Dist``     | obsoleted by the distribution                         | | ``Requires-Dist: zope.interface (>3.5.0)`` |
  | | ``Obsoletes-Dist``    |                                                       | | ``Provides-Dist: virtual_package``         |
  |                         |                                                       | | ``Obsoletes-Dist: OtherProject (<3.0)``    |
  +-------------------------+-------------------------------------------------------+----------------------------------------------+

* Other new fields: ``Project-URL``, ``Maintainer``, and ``Maintainer-email``


PEP 386 -- "Changing the version comparison module in Distutils"
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* Goal: Specify a common standard for version numbers

* Motivation:

  - Without a common standard, installers (e.g., ``pip``) have a hard time
    knowing which version of a package to install

* Pseudo-format: ``N.N[.N]+[{a|b|c|rc}N[.N]+][.postN][.devN]``

* Examples (in increasing order)::

    1.0a1           # alpha
    1.0a2.1         # alpha
    1.0b2           # beta
    1.0c1           # release candidate
    1.0rc2          # release candidate
    1.0.dev456      # development
    1.0             # final
    1.0.post456     # post-release

* To support interoperability with other versioning schemes, a
  ``suggest_normalized_version()`` method is provided to turn a non-compliant
  version into a compliant version


New Packaging Library: ``packaging``/``distutils2``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* Goal: Provide a new standard packaging library that implements the new
  standards and provides additional improvements over the current libraries

* Motivation:

  - ``setuptools`` made some valuable improvements, but it is still built on
    top of the flawed design of ``distutils``

  - Redesigning ``distutils`` would break all the current tools; a new library
    was needed to implement the new standards

* The new library will be available as ``packaging`` in the Python 3.3+
  standard library and ``distutils2`` on PyPI for Python 2.4-3.2

* Obsoletes older packaging libraries: ``distutils``, ``setuptools``, and
  ``distribute``

* Backward-compatible: Supports browsing and installing distributions packaged
  with older libraries

* Static configuration file (``setup.cfg``) replaces ``setup.py``

  - Motivation:

    + ``setup.py`` requires running code on the target platform just to read a
      distribution's metadata; the generated metadata is specific to the
      platform where ``setup.py`` was executed

    + Package managers must download a distribution to determine its
      requirements and build a dependency graph

  - Sample ``setup.cfg``::

     [metadata]
     name = sample
     version = 1.1.0
     requires-python = >=2.4, <3.2
     requires-dist = pywin32; sys.platform == 'win32'
     # ...

     [files]
     packages = sample1
     # ...

  - The only reason to provide a ``setup.py`` is backwards compatibility

* Improved description of a distribution's data files:

  - OS packagers can tell the difference between types of data files
    (documentation, configuration files, etc.) and install them in the
    appropriate locations

* Hooks are provided to run additional code before/after commands (e.g., to
  perform additional checks prior to running the *install* command)

* Usable as a library. Provides implementations of the new standards:

  - PEP 376 in ``packaging.database``

  - PEP 345 in ``packaging.metadata``

  - PEP 386 in ``packaging.version``


Tool Support
------------

* ``pysetup``

  - Barebones command-line tool shipped with ``packaging``/``distutils2``

* ``pip``

  - Will be updated to use ``packaging``/``distutils`` instead of
    ``setuptools``

  - Will continue to provide additional functionality (SCM support,
    requirements files, etc.)
