==============================
Using a shared build directory
==============================

For builds that take a long time, it can be convenient to reuse them across
several buildouts. To do this, use the `shared` option:

    >>> cache = tmpdir('cache')
    >>> write('buildout.cfg',
    ... """
    ... [buildout]
    ... parts = foo
    ... download-cache = %s
    ...
    ... [foo]
    ... recipe = zc.recipe.cmmi
    ... url = file://%s/foo.tgz
    ... shared = True
    ... """ % (cache, distros))

When run the first time, the build is executed as usual:

    >>> print(system('bin/buildout'))
    Installing foo.
    foo: Unpacking and configuring
    configuring foo /cache/cmmi/build/<BUILDID>
    echo make is called with option 
    make is called with option
    echo building foo
    building foo
    echo installing foo
    installing foo
    <BLANKLINE>

But after that, the existing shared build directory is used instead of running
the build again:

    >>> remove('.installed.cfg')
    >>> print(system('bin/buildout'))
    Installing foo.
    foo: using existing shared build
    <BLANKLINE>


The shared directory
====================

By default, the shared build directory is named with a hash of the recipe's
configuration options (but it can also be configured manually, see below):

    >>> ls(cache, 'cmmi', 'build')
    d  <BUILDID>

For example, if the download url changes, the build is executed again:

    >>> import os
    >>> import shutil
    >>> _ = shutil.copy(os.path.join(distros, 'foo.tgz'),
    ...                 os.path.join(distros, 'qux.tgz'))

    >>> remove('.installed.cfg')
    >>> write('buildout.cfg',
    ... """
    ... [buildout]
    ... parts = qux
    ... download-cache = %s
    ...
    ... [qux]
    ... recipe = zc.recipe.cmmi
    ... url = file://%s/qux.tgz
    ... shared = True
    ... """ % (cache, distros))
    >>> print(system('bin/buildout'))
    Installing qux.
    qux: Unpacking and configuring
    configuring foo /cache/cmmi/build/<BUILDID>
    echo make is called with option 
    make is called with option
    echo building foo
    building foo
    echo installing foo
    installing foo

and another shared directory is created:

    >>> ls(cache, 'cmmi', 'build')
    d  <BUILDID>
    d  <BUILDID>

(Other recipes can retrieve the shared build directory from our part's
`location` as usual, so the SHA-names shouldn't be a problem.)


Configuring the shared directory
================================

If you set `shared` to an existing directory, that will be used as the build
directory directly (instead of a name computed from to the recipe options):

    >>> shared = os.path.join(cache, 'existing')
    >>> os.mkdir(shared)
    >>> write('buildout.cfg',
    ... """
    ... [buildout]
    ... parts = foo
    ...
    ... [foo]
    ... recipe = zc.recipe.cmmi
    ... url = file://%s/foo.tgz
    ... shared = %s
    ... """ % (distros, shared))

    >>> remove('.installed.cfg')
    >>> print(system('bin/buildout'))
    Installing foo.
    foo: Unpacking and configuring
    configuring foo /cache/existing/cmmi
    echo make is called with option 
    make is called with option
    echo building foo
    building foo
    echo installing foo
    installing foo
    <BLANKLINE>

If no download-cache is set, and `shared` is not a directory, an error is raised:

    >>> write('buildout.cfg',
    ... """
    ... [buildout]
    ... parts = foo
    ...
    ... [foo]
    ... recipe = zc.recipe.cmmi
    ... url = file://%s/foo.tgz
    ... shared = True
    ... """ % distros)

    >>> print(system('bin/buildout').strip())
    While:
      Installing.
      Getting section foo.
      Initializing section foo.
    ...
    ValueError: Set the 'shared' option of zc.recipe.cmmi to an existing
    directory, or set ${buildout:download-cache}


Build errors
============

If an error occurs during the build (or it is aborted by the user),
the build directory is removed, so there is no risk of accidentally
mistaking some half-baked build directory as a good cached shared build.

Let's simulate a build error. First, we backup a working build.

    >>> _ = shutil.copy(os.path.join(distros, 'foo.tgz'),
    ...                 os.path.join(distros, 'foo.tgz.bak'))

Then we create a broken tarball:

    >>> import tarfile
    >>> from zc.recipe.cmmi.tests import BytesIO
    >>> import sys
    >>> tarpath = os.path.join(distros, 'foo.tgz')
    >>> with tarfile.open(tarpath, 'w:gz') as tar:
    ...    configure = 'invalid'
    ...    info = tarfile.TarInfo('configure.off')
    ...    info.size = len(configure)
    ...    info.mode = 0o755
    ...    tar.addfile(info, BytesIO(configure))

Now we reset the cache to force our broken tarball to be used:

    >>> shutil.rmtree(cache)
    >>> cache = tmpdir('cache')
    >>> write('buildout.cfg',
    ... """
    ... [buildout]
    ... parts = foo
    ... download-cache = %s
    ...
    ... [foo]
    ... recipe = zc.recipe.cmmi
    ... url = file://%s/foo.tgz
    ... shared = True
    ... """ % (cache, distros))

    >>> remove('.installed.cfg')
    >>> res = system('bin/buildout')
    >>> print(res)
    Installing foo.
    ...
    ValueError: Couldn't find configure

The temporary directory where tarball was unpacked was left behind for
debugging purposes.

    >>> import re
    >>> shutil.rmtree(re.search('foo: cmmi failed: (.*)', res).group(1))

When we now fix the error (by copying back the working version and resetting the
cache), the build will be run again, and we don't use a half-baked shared
directory:

    >>> _ = shutil.copy(os.path.join(distros, 'foo.tgz.bak'),
    ...                 os.path.join(distros, 'foo.tgz'))
    >>> shutil.rmtree(cache)
    >>> cache = tmpdir('cache')
    >>> write('buildout.cfg',
    ... """
    ... [buildout]
    ... parts = foo
    ... download-cache = %s
    ...
    ... [foo]
    ... recipe = zc.recipe.cmmi
    ... url = file://%s/foo.tgz
    ... shared = True
    ... """ % (cache, distros))
    >>> print(system('bin/buildout'))
    Installing foo.
    foo: Unpacking and configuring
    configuring foo /cache/cmmi/build/<BUILDID>
    echo make is called with option 
    make is called with option
    echo building foo
    building foo
    echo installing foo
    installing foo
    <BLANKLINE>


Interaction with other users of shared builds
=============================================

While shared builds are a way to cache a build between installation runs of a
given buildout part, they are, more importantly, shared between multiple parts
and most probably, multiple buildouts. This implies two general rules of
behaviour: We should never delete shared builds, and we need to be prepared
for shared builds to be deleted by other system at any time.

In other words: Every install or update run of the recipe that uses a shared
build needs to check whether the build still exists on disk and rebuild it if
it does not. On the other hand, a part using the shared build must not declare
the shared build its own property lest buildout remove it when the shared
build is no longer needed, either because the part no longer uses it or
because the part itself is no longer used.

The last thing we did above was to install a shared build:

    >>> ls(cache, 'cmmi', 'build')
    d  <BUILDID>

If someone deletes this shared build, updating the buildout part that needs it
will cause it to be rebuilt:

    >>> rmdir(cache, 'cmmi', 'build')
    >>> print(system('bin/buildout').strip())
    Updating foo.
    foo: Unpacking and configuring
    configuring foo /cache/cmmi/build/<BUILDID>
    echo make is called with option 
    make is called with option
    echo building foo
    building foo
    echo installing foo
    installing foo

    >>> ls(cache, 'cmmi', 'build')
    d  <BUILDID>

If we stop using the shared build, it stays in the build cache:

    >>> write('buildout.cfg',
    ... """
    ... [buildout]
    ... parts = foo
    ... download-cache = %s
    ...
    ... [foo]
    ... recipe = zc.recipe.cmmi
    ... url = file://%s/foo.tgz
    ... """ % (cache, distros))

    >>> print(system('bin/buildout').strip())
    Uninstalling foo.
    Installing foo.
    foo: Unpacking and configuring
    configuring foo /sample-buildout/parts/foo
    echo make is called with option 
    make is called with option
    echo building foo
    building foo
    echo installing foo
    installing foo

    >>> ls(cache, 'cmmi', 'build')
    d  <BUILDID>


Regression: Keeping track of a reused shared build
==================================================

Let's first remove and rebuild everything to get some measure of isolation
from the story so far:

    >>> remove('.installed.cfg')
    >>> rmdir(cache, 'cmmi', 'build')

    >>> write('buildout.cfg',
    ... """
    ... [buildout]
    ... parts = foo
    ... download-cache = %s
    ...
    ... [foo]
    ... recipe = zc.recipe.cmmi
    ... url = file://%s/foo.tgz
    ... shared = True
    ... """ % (cache, distros))

    >>> print(system('bin/buildout'))
    Installing foo.
    foo: Unpacking and configuring
    configuring foo /cache/cmmi/build/<BUILDID>
    echo make is called with option 
    make is called with option
    echo building foo
    building foo
    echo installing foo
    installing foo

zc.recipe.cmmi 1.2 had a bug that manifested after reusing a shared build: The
part wouldn't keep track of the shared build and thus wasn't able to restore
it if it got deleted from the cache. This is how it should work:

    >>> remove('.installed.cfg')
    >>> print(system('bin/buildout'))
    Installing foo.
    foo: using existing shared build

    >>> rmdir(cache, 'cmmi', 'build')
    >>> print(system('bin/buildout').strip())
    Updating foo.
    foo: Unpacking and configuring
    configuring foo /cache/cmmi/build/<BUILDID>
    echo make is called with option 
    make is called with option
    echo building foo
    building foo
    echo installing foo
    installing foo


Regression: Don't leave behind a build directory if the download failed
=======================================================================

zc.recipe.cmmi up to version 1.3.1 had a bug that caused an empty build
directory to be left behind if a download failed, causing it to be mistaken
for a good shared build.

We cause the download to fail by specifying a nonsensical MD5 sum:

    >>> shutil.rmtree(cache)
    >>> cache = tmpdir('cache')
    >>> write('buildout.cfg',
    ... """
    ... [buildout]
    ... parts = foo
    ... download-cache = %s
    ...
    ... [foo]
    ... recipe = zc.recipe.cmmi
    ... url = file://%s/foo.tgz
    ... md5sum = 1234
    ... shared = True
    ... """ % (cache, distros))

    >>> remove('.installed.cfg')
    >>> print(system('bin/buildout'))
    Installing foo.
    ...
    Error: MD5 checksum mismatch for local resource at '/distros/foo.tgz'.

The build directory must not exist anymore:

    >>> ls(cache, 'cmmi')

Another buildout run must fail the same way as the first attempt:

    >>> print(system('bin/buildout'))
    Installing foo.
    ...
    Error: MD5 checksum mismatch for local resource at '/distros/foo.tgz'.
