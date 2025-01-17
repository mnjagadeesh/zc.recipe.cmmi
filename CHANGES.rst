=================
 Release History
=================

3.1.0 (unreleased)
==================

- Add support for Python 3.9.


3.0.0 (2019-03-30)
==================

- Drop support for Python 3.4.

- Add support for Python 3.7 and 3.8a2.

- Flake8 the code.


2.0.0 (2017-06-21)
==================

- Add support for Python 3.4, 3.5, 3.6 and PyPy.

- Automated testing is enabled on Travis CI.

1.3.6 (2014-04-14)
==================

- Fixed: Strings were incorrectly compared using "is not ''" rather than !=

- Fixed: Spaces weren't allowed in the installation location.

1.3.5 (2011-08-06)
==================

- Fixed: Spaces weren't allowed in environment variables.

- Fixed: Added missing option reference documentation.


1.3.4 (2011-01-18)
==================

- Fixed a bug in location book-keeping that caused shared builds to be deleted
  from disk when a part didn't need them anymore. (#695977)

- Made tests pass with both zc.buildout 1.4 and 1.5, lifted the upper version
  bound on zc.buildout. (#695732)

1.3.3 (2010-11-10)
==================

- Remove the temporary build directory when cmmi succeeds.

- Specify that the zc.buildout version be <1.5.0b1, as the recipe is
  currently not compatible with zc.buildout 1.5.

1.3.2 (2010-08-09)
==================

- Remove the build directory for a shared build when the source cannot be
  downloaded.

- Declared a test dependency on zope.testing.


1.3.1 (2009-09-10)
==================

- Declare dependency on zc.buildout 1.4 or later. This dependency was introduced
  in version 1.3.


1.3 (2009-09-03)
================

- Use zc.buildout's download API. As this allows MD5 checking, added the
  md5sum and patch-md5sum options.

- Added options for changing the name of the configure script and
  overriding the ``--prefix`` parameter.

- Moved the core "configure; make; make install" command sequence to a
  method that can be overridden in other recipes, to support packages
  whose installation process is slightly different.

1.2.1 (2009-08-12)
==================

Bug fix: keep track of reused shared builds.


1.2.0 (2009-05-18)
==================

Enabled using a shared directory for completed builds.

1.1.6 (2009-03-17)
==================

Moved 'zc' package from root of checkout into 'src', to prevent testrunner
from finding eggs installed locally by buildout.

Removed deprecations under Python 2.6.

1.1.5 (2008-11-07)
==================

- Added to the README.txt file a link to the SVN repository, so that Setuptools
  can automatically find the development version when asked to install the
  "-dev" version of zc.recipe.cmmi.

- Applied fix for bug #261367 i.e. changed open() of file being downloaded to
  binary, so that errors like the following no longer occur under Windows.

  uncompress = self.decompress.decompress(buf)
  error: Error -3 while decompressing: invalid distance too far back

1.1.4 (2008-06-25)
==================

Add support to autogen configure files.

1.1.3 (2008-06-03)
==================

Add support for updating the environment.

1.1.2 (2008-02-28)
==================

- Check if the ``location`` folder exists before creating it.

After 1.1.0
===========

Added support for patches to be downloaded from a url rather than only using
patches on the filesystem

1.1.0
=====

Added support for:

 - download-cache: downloaded files are cached in the 'cmmi' subdirectory of
   the cache cache keys are hashes of the url that the file was downloaded from
   cache information recorded in the cache.ini file within each directory

 - offline mode: cmmi will not go online if the package is not in the cache

 - variable location: build files other than in the parts directory if required

 - additional logging/output

1.0.2 (2007-06-03)
==================

- Added support for patches.

- Tests fixed (buildout's output changed)

1.0.1 (2006-11-22)
==================

- Added missing zip_safe flag.

1.0 (2006-11-22)
================

Initial release.
