Loading Patces from URLs
========================

Patch files can be loaded from URLs as well as files.

Any downloaded patch files can be cached in a download cache if
available, in exactly the same way as for tarballs.  Similarly,
if the build is set to offline operation, then it will not download
from a remote location.

To see how this works, we'll set up a web server with a patch file,
and a cache with our tarball in:

    >>> import sys, os
    >>> cache = tmpdir('cache')
    >>> patch_data = tmpdir('patch_data')
    >>> patchfile = os.path.join(patch_data, 'config.patch')
    >>> write(patchfile,
    ... """--- configure
    ... +++ /dev/null
    ... @@ -1,13 +1,13 @@
    ...  #!%s
    ...  import sys
    ... -print("configuring foo " + ' '.join(sys.argv[1:]))
    ... +print("configuring foo patched " + ' '.join(sys.argv[1:]))
    ...
    ...  Makefile_template = '''
    ...  all:
    ... -\techo building foo
    ... +\techo building foo patched
    ...
    ...  install:
    ... -\techo installing foo
    ... +\techo installing foo patched
    ...  '''
    ...
    ...  open('Makefile', 'w').write(Makefile_template)
    ...
    ... """ % sys.executable)

    >>> server_url = start_server(patch_data)

Now let's create a buildout.cfg file.

    >>> write('buildout.cfg',
    ... """
    ... [buildout]
    ... parts = foo
    ... download-cache=%(cache)s
    ... log-level = DEBUG
    ...
    ... [foo]
    ... recipe = zc.recipe.cmmi
    ... url = file://%(distros)s/foo.tgz
    ... patch = %(server_url)s/config.patch
    ... """ % dict(distros=distros,server_url=server_url,cache=cache))

    >>> print(system('bin/buildout').strip())
    In...
    Installing foo.
    foo: Searching cache at /cache/cmmi
    foo: Cache miss; will cache /distros/foo.tgz as /cache/cmmi/...
    foo: Using local resource /distros/foo.tgz
    foo: Unpacking and configuring
    foo: Searching cache at /cache/cmmi
    foo: Cache miss; will cache http://localhost//config.patch as /cache/cmmi/...
    foo: Downloading http://localhost//config.patch
    patching file configure
    ...
    configuring foo patched /sample-buildout/parts/foo
    echo make is called with option 
    make is called with option
    echo building foo patched
    building foo patched
    echo installing foo patched
    installing foo patched

Any downloaded patch files can be cached in a download cache if available, in
exactly the same way as for tarballs.  Similarly, if the build is set to offline
operation, then it will not download from a remote location.

We can see that the patch is now in the cache, as well as the tarball:

    >>> import os
    >>> cache_path = os.path.join(cache, 'cmmi')
    >>> ls(cache_path)
    - ...
    - ...
