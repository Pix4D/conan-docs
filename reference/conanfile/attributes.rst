Attributes
==========

name
----
This is a string, with a minimun of 2 and a maximum of 50 characters (though shorter names are recommended), that defines the package name. It will be the ``<PkgName>/version@user/channel`` of the package reference.
It should match the following regex ``^[a-zA-Z0-9_][a-zA-Z0-9_\+\.-]$``, so start with alphanumeric or underscore, then alphanumeric, underscore, +, ., - characters.

The name is only necessary for ``export``-ing the recipe into the local cache (``export`` and ``create`` commands), if they are not defined in the command line.
It might take its value from an environment variable, or even any python code that defines it (e.g. a function that reads an environment variable, or a file from disk). 
However, the most common and suggested approach would be to define it in plain text as a constant, or provide it as command line arguments.


version
-------
The version attribute will define the version part of the package reference: ``PkgName/<version>@user/channel``
It is a string, and can take any value, matching the same constraints as the ``name`` attribute.
In case the version follows semantic versioning in the form ``X.Y.Z-pre1+build2``, that value might be used for requiring this package through version ranges instead of exact versions.

The version is only strictly necessary for ``export``-ing the recipe into the local cache (``export`` and ``create`` commands), if they are not defined in the command line.
It might take its value from an environment variable, or even any python code that defines it (e.g. a function that reads an environment variable, or a file from disk).
Please note that this value might be used in the recipe in other places (as in ``source()`` method to retrieve code from elsewhere), making this value not constant means that it may evaluate differently in different contexts (e.g., on different machines or for different users) leading to unrepeatable or unpredictable results.
The most common and suggested approach would be to define it in plain text as a constant, or provide it as command line arguments.


description
------------
This is an optional, but strongly recommended text field, containing the description of the package,
and any information that might be useful for the consumers. The first line might be used as a
short description of the package.


.. code-block:: python

   class HelloConan(ConanFile):
       name = "Hello"
       version = "0.1"
       description = """This is a Hello World library.
                        A fully featured, portable, C++ library to say Hello World in the stdout,
                        with incredible iostreams performance"""


.. _package_url:

url
---

It is possible, even typical, if you are packaging a thid party lib, that you just develop
the packaging code. Such code is also subject to change, often via collaboration, so it should be stored
in a VCS like git, and probably put on GitHub or a similar service. If you do indeed maintain such a
repository, please indicate it in the ``url`` attribute, so that it can be easily found.

.. code-block:: python

   class HelloConan(ConanFile):
       name = "Hello"
       version = "0.1"
       url = "https://github.com/memsharded/hellopack.git"


The ``url`` is the url **of the package** repository, i.e. not necessarily the original source code.
It is optional, but highly recommended, that it points to GitHub, Bitbucket or your preferred
code collaboration platform. Of course, if you have the conanfile inside your library source,
you can point to it, and afterwards use the ``url`` in your ``source()`` method.

This is a recommended, but not mandatory attribute.

license
---------
This field is intended for the license of the **target** source code and binaries, i.e. the code
that is being packaged, not the ``conanfile.py`` itself. This info is used to be displayed by
the ``conan info`` command and possibly other search and report tools.

.. code-block:: python

   class HelloConan(ConanFile):
       name = "Hello"
       version = "0.1"
       license = "MIT"

This attribute can contain several, comma separated licenses. It is a text string, so it can
contain any text, including hyperlinks to license files elsewhere.

This is a recommended, but not mandatory attribute.

author
------

Intended to add information about the author, in case it is different from the conan user. It is
possible that the conan user is the name of an organization, project, company or group, and many
users have permissions over that account. In this case, the author information can explicitely
define who is the creator/maintainer of the package

.. code-block:: python

   class HelloConan(ConanFile):
       name = "Hello"
       version = "0.1"
       author = "John J. Smith (john.smith@company.com)"

This is an optional attribute

.. _user_channel:

user, channel
--------------

The fields ``user`` and ``channel`` can be accessed from within a ``conanfile.py``.
Though their usage is usually not encouraged, it could be useful in different cases,
e.g. to define requirements with the same user and
channel than the current package, which could be achieved with something like:

.. code-block:: python

    from conans import ConanFile

    class HelloConan(ConanFile):
        name = "Hello"
        version = "0.1"

        def requirements(self):
            self.requires("Say/0.1@%s/%s" % (self.user, self.channel))


Only package recipes that are in the conan local cache (i.e. "exported") have an user/channel assigned.
For package recipes working in user space, there is no current user/channel. The properties ``self.user``
and ``self.channel`` will then look for environment variables ``CONAN_USERNAME`` and ``CONAN_CHANNEL``
respectively. If they are not defined, an error will be raised.


.. _settings_property:

settings
----------

There are several things that can potentially affect a package being created, i.e. the final
package will be different (a different binary, for example), if some input is different.

Development project-wide variables, like the compiler, its version, or the OS
itself. These variables have to be defined, and they cannot have a default value listed in the
conanfile, as it would not make sense.

It is obvious that changing the OS produces a different binary in most cases. Changing the compiler
or compiler version changes the binary too, which might have a compatible ABI or not, but the
package will be different in any case.

But what happens for example to **header only libraries**? The final package for such libraries is not
binary and, in most cases it will be identical, unless it is automatically generating code.
We can indicate that in the conanfile:

.. code-block:: python

   from conans import ConanFile

   class HelloConan(ConanFile):
       name = "Hello"
       version = "0.1"
       # We can just omit the settings attribute too
       settings = None

       def build(self):
            #empty too, nothing to build in header only


You can restrict existing settings and accepted values as well, by redeclaring the settings
attribute:

.. code-block:: python

   class HelloConan(ConanFile):
      settings = {"os": ["Windows"],
                  "compiler": {"Visual Studio": {"version": [11, 12]}},
                  "arch": None}

In this example we have just defined that this package only works in Windows, with VS 10 and 11.
Any attempt to build it in other platforms with other settings will throw an error saying so.
We have also defined that the runtime (the MD and MT flags of VS) is irrelevant for us
(maybe we using a universal one?). Using None as a value means, *maintain the original values* in order
to avoid re-typing them. Then, "arch": None is totally equivalent to "arch": ["x86", "x86_64", "arm"]
Check the reference or your ~/.conan/settings.yml file.

As re-defining the whole settings attribute can be tedious, it is sometimes much simpler to
remove or tune specific fields in the ``config()`` method. For example, if our package is runtime
independent in VS, we can just remove that setting field:


.. code-block:: python

   settings = "os", "compiler", "build_type", "arch"

   def config(self):
       self.settings.compiler["Visual Studio"].remove("runtime")

.. _conanfile_options:

options, default_options
---------------------------
Conan packages recipes can generate different package binaries when different settings are used, but can also customize, per-package any other configuration that will produce a different binary.

A typical option would be being shared or static for a certain library. Note that this is optional, different packages can have this option, or not (like header-only packages), and different packages can have different values for this option, as opposed to settings, which typically have the same values for all packages being installed (though this can be controlled too, defining different settings for specific packages)

Options are defined in package recipes as dictionaries of name and allowed values:

.. code-block:: python

    class MyPkg(ConanFile):
        ...
        options = {"shared": [True, False]}

There is an special value ``ANY`` to allow any value for a given option. The range of values for such an option will not be checked, and any value (as string) will be accepted:

.. code-block:: python

    class MyPkg(ConanFile):
        ...
        options = {"shared": [True, False], "commit": "ANY"}


When a package is installed, it will need all its options be defined a value. Those values can be defined in command line, profiles, but they can also (and they will be typically) defined in conan package recipes:

.. code-block:: python

    class MyPkg(ConanFile):
        ...
        options = {"shared": [True, False], "fPIC": [True, False]}
        default_options = "shared=False", "fPIC=False"

The options will typically affect the ``build()`` of the package in some way, for example:

.. code-block:: python

   class MyPkg(ConanFile):
      ...
      options = {"shared": [True, False]}
      default_options = "shared=False"

      def build(self):
         shared = "-DBUILD_SHARED_LIBS=ON" if self.options.shared else ""
         cmake = CMake(self)
         self.run("cmake . %s %s" % (cmake.command_line, shared))
         self.run("cmake --build . %s" % cmake.build_config)

Note that you have to consider the option properly in your build scripts. In this case, we are using the CMake way. So if you had explicit **STATIC** linkage in the **CMakeLists.txt** file, you have to remove it. If you are using VS, you also need to change your code to correctly import/export symbols for the dll.

This is only an example. Actually, the ``CMake`` helper already automates this, so it is enough to do:

.. code-block:: python

    def build(self):
        cmake = CMake(self) # internally it will check self.options.shared
        self.run("cmake . %s" % cmake.command_line) # or cmake.configure()
        self.run("cmake --build . %s" % cmake.build_config) # or cmake.build()


You can also specify default option values of the required dependencies:

.. code-block:: python

   class OtherPkg(ConanFile):
      requires = "Pkg/0.1@user/channel"
      default_options = "Pkg:pkg_option=value"

If you need to dynamically set some dependency options, you could do:

.. code-block:: python

   class OtherPkg(ConanFile):
      requires = "Pkg/0.1@user/channel"

      def configure(self):
          self.options["Pkg"].pkg_option = "value"


Option values can be given in command line, and they will have priority over the default values in the recipe:

.. code-block:: bash

    $ conan install -o Pkg:shared=True -o OtherPkg:option=value

You can also defined them in consumer ``conanfile.txt``, as described in :ref:`this section<options_txt>`

.. code-block:: text

    [requires]
    Poco/1.7.8p3@pocoproject/stable

    [options]
    Poco:shared=True
    OpenSSL:shared=True

And finally, you can define options in :ref:`profiles<profiles>` too, with the same syntax:

.. code-block:: text

    # file "myprofile"
    # use it as $ conan install -pr=myprofile
    [settings]
    setting=value

    [options]
    MyLib:shared=True


You can inspect available package options, reading the package recipe, which is conveniently done with:

.. code-block:: bash

    $ conan get Pkg/0.1@user/channel

requires
---------

Specify package dependencies as a list of other packages:


.. code-block:: python

   class MyLibConan(ConanFile):
       requires = "Hello/1.0@user/stable", "OtherLib/2.1@otheruser/testing"

You can specify further information about the package requirements:

.. code-block:: python

   class MyLibConan(ConanFile):
      requires = (("Hello/0.1@user/testing"),
                  ("Say/0.2@dummy/stable", "override"),
                  ("Bye/2.1@coder/beta", "private"))

Requirements can be complemented by 2 different parameters:

**private**: a dependency can be declared as private if it is going to be fully embedded and hidden
from consumers of the package. Typical examples could be a header only library which is not exposed
through the public interface of the package, or the linking of a static library inside a dynamic
one, in which the functionality or the objects of the linked static library are not exposed through
the public interface of the dynamic library.

**override**: packages can define overrides of their dependencies, if they require the definition of
specific versions of the upstream required libraries, but not necessarily direct dependencies. For example,
a package can depend on A(v1.0), which in turn could conditionally depend on Zlib(v2), depending on whether
the compression is enabled or not. Now, if you want to force the usage of Zlib(v3) you can:

..  code-block:: python

   class HelloConan(ConanFile):
      requires = ("A/1.0@user/stable", ("Zlib/3.0@other/beta", "override"))


This **will not introduce a new dependency**, it will just change Zlib v2 to v3 if A actually
requires it. Otherwise Zlib will not be a dependency of your package.

.. _version_ranges_reference:

version ranges
++++++++++++++

The syntax is using brackets:

..  code-block:: python

   class HelloConan(ConanFile):
      requires = "Pkg/[>1.0,<1.8]@user/stable"

Expressions are those defined and implemented by [python node-semver](https://pypi.python.org/pypi/node-semver),
but using a comma instead of spaces. Accepted expressions would be:

..  code-block:: python

   >1.1,<2.1    # In such range
   2.8          # equivalent to =2.8
   ~=3.0        # compatible, according to semver
   >1.1 || 0.8  # conditions can be OR'ed


.. container:: out_reference_box

    Go to :ref:`Mastering/Version Ranges<version_ranges>` if you want to learn more about version ranges.

build_requires
----------------

Build requirements are requirements that are only installed and used when the package is built from sources. If there is an existing pre-compiled binary, then the build requirements for this package will not be retrieved.

They can be specified as a comma separated tuple in the package recipe:

.. code-block:: python

    class MyPkg(ConanFile):
        build_requires = "ToolA/0.2@user/testing", "ToolB/0.2@user/testing"

Read more: :ref:`Build requiremens <build_requires>`


exports
--------
If a package recipe ``conanfile.py`` requires other external files, like other python files that
it is importing (python importing), or maybe some text file with data it is reading, those files
must be exported with the ``exports`` field, so they are stored together, side by side with the
``conanfile.py`` recipe.

The ``exports`` field can be one single pattern, like ``exports="*"``, or several inclusion patterns.
For example, if we have some python code that we want the recipe to use in a ``helpers.py`` file,
and have some text file, ``info.txt``, we want to read and display during the recipe evaluation
we would do something like:

.. code-block:: python

   exports = "helpers.py", "info.txt"

Exclude patterns are also possible, with the ``!`` prefix:

.. code-block:: python

   exports = "*.py", "!*tmp.py"


This is an optional attribute, only to be used if the package recipe requires these other files
for evaluation of the recipe.

exports_sources
----------------
There are 2 ways of getting source code to build a package. Using the ``source()`` recipe method
and using the ``exports_sources`` field. With ``exports_sources`` you specify which sources are required,
and they will be exported together with the **conanfile.py**, copying them from your folder to the
local conan cache. Using ``exports_sources``
the package recipe can be self-contained, containing the source code like in a snapshot, and then
not requiring downloading or retrieving the source code from other origins (git, download) with the
``source()`` method when it is necessary to build from sources.

The ``exports_sources`` field can be one single pattern, like ``exports_sources="*"``, or several inclusion patterns.
For example, if we have the source code inside "include" and "src" folders, and there are other folders
that are not necessary for the package recipe, we could do:

.. code-block:: python

   exports_sources = "include*", "src*"

Exclude patterns are also possible, with the ``!`` prefix:

.. code-block:: python

   exports_sources = "include*", "src*", "!src/build/*"

This is an optional attribute, used typically when ``source()`` is not specify. The main difference with
``exports`` is that ``exports`` files are always retrieved (even if pre-compiled packages exist),
while ``exports_sources`` files are only retrieved when it is necessary to build a package from sources.

generators
----------

Generators specify which is the output of the ``install`` command in your project folder. By
default, a ``conanbuildinfo.txt`` file is generated, but you can specify different generators.

Check the full generators list in :ref:`Reference/Generators<generators>`

You can specify more than one generator:

.. code-block:: python

   class MyLibConan(ConanFile):
       generators = "cmake", "gcc"


build_policy
------------

With the ``build_policy`` attribute the package creator can change the default conan's build behavior.
The allowed ``build_policy`` values are:

- ``missing``: If no binary package is found, conan will build it without the need of invoke conan install with **--build missing** option.
- ``always``: The package will be built always, **retrieving each time the source code** executing the "source" method.


.. code-block:: python
   :emphasize-lines: 2

     class PocoTimerConan(ConanFile):
        build_policy = "always" # "missing"

short_paths
------------

If one of the packages you are creating hits the limit of 260 chars path length in Windows, add
``short_paths=True`` in your conanfile.py:

..  code-block:: python

   from conans import ConanFile

   class ConanFileTest(ConanFile):
       ...
       short_paths = True

This will automatically "link" the ``source`` and ``build`` directories of the package to the drive root,
something like `C:/.conan/tmpdir`. All the folder layout in the conan cache is maintained.

This attribute will not have any effect in other OS, it will be discarded.

From Windows 10 (ver. 10.0.14393), it is possible to opt-in disabling the path limits. Check `this link
<https://msdn.microsoft.com/en-us/library/windows/desktop/aa365247(v=vs.85).aspx#maxpath>`_ for more info. Latest python installers might offer to enable this while installing python. With this limit removed, the ``short_paths`` functionality is totally unnecessary.


no_copy_source
---------------

The attribute ``no_copy_source`` tells the recipe that the source code will not be copied from the ``source`` folder to the ``build`` folder. 
This is mostly an optimization for packages with large source codebases, to avoid extra copies. It is **mandatory** that the source code must not be modified at all by the configure or build scripts, as the source code will be shared among all builds.

To be able to use it, the package recipe can access the ``self.source_folder`` attribute, which will point to the ``build`` folder when ``no_copy_source=False`` or not defined, and will point to the ``source`` folder when ``no_copy_source=True``

When this attribute is set to True, the ``package()`` method will be called twice, one copying from the ``source`` folder and the other copying from the ``build`` folder.


folders
---------
In the package recipe methods, some attributes pointing to the relevant folders can be defined. Not all of them will be defined always, only in those relevant methods that might use them.

- ``self.source_folder``: the folder in which the source code to be compiled lives. When a package is built in the conan local cache, by default it is the ``build`` folder, as the source code is copied from the ``source`` folder to the ``build`` folder, to ensure isolation and avoiding modifications of shared common source code among builds for different configurations. Only when ``no_copy_source=True`` this folder will actually point to the package ``source`` folder in the local cache.
- ``self.build_folder``: the folder in which the build is being done
- ``self.package_folder``: the folder to copy the final artifacts for the package binary

When executing local conan commands (for a package not in the local cache, but in user folder), those fields would be pointing to the corresponding local user folder.


conanfile_directory
-------------------

``self.conanfile_directory`` is a **read only property** that returns the directory in which the conanfile is
located.


cpp_info
---------
This attribute is only defined inside ``package_info()`` method, being None elsewhere, so please use it only inside this method.
Read :ref:`package_info() method docs <package_info>` for more info.

