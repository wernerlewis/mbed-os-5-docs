# Migration guide

<style> 
#dep {background-color:red; color:black}
#mod {background-color:orange; color:black}
#new {background-color:green; color:black}
</style>

If you haven't already read the [introduction](./intro.md), then start from there.

The background color indicates:

* <span id=new>Green: New Functionality introduced in Mbed CLI 2 (mbed-tools)</span>
* <span id=mod>Orange: Functionality modified in Mbed CLI2 (mbed-tools) compared to Mbed CLI 1 (mbed-cli)</span>
* <span id=dep>Red: Functionality deprecated in Mbed CLI 2 (mbed-tools)</span>

## Installers
<span id=mod>Windows, MacOS and Linux installers will be made available at later point of time.</span></br>

## Commands

### Device management
<span id=dep>The subcommand "mbed-cli device-management (device management) is deprecated. Refer to your cloud provider's documentation on how to do this.</span></br>

### Repository management
<span id=dep>Support for "Mercurial" and the subcommand "mbed-cli publish (Publish program or library)" is deprecated. Due to this, hosted repositories on "mbed.org" is not supported anymore. "Git" can be used as an alternative.</span></br>

<span id=dep>Support for subcommand "mbed-cli cache (Repository cache management)" is deprecated.</span></br>

<span id=dep>Support for subcommand "mbed-cli releases (Show release tags)" is deprecated. Use standard "git" commands instead.</span></br>

<span id=mod>The subcommand "mbed-cli update (Update to branch, tag, revision or latest)" is not implemented. Use standard "git" commands instead.</span></br>

<span id=dep>The subcommand "mbed-cli export (Generate an IDE project)" is deprecated. Use "CMake" exporters instead.</span></br>

### Library management
<span id=dep>mbed-cli add</span>
Adding a library from source control by URL is deprecated in Mbed CLI 2. To add a library, use `mbed-tools import URL SUBDIR` where `SUBDIR` is the library subdirectory, then use `git add SUBDIR`. Various wrapper functions around `git` have been removed, to improve usability, maintainability, and to encourage direct use with more control.

<span id=dep>mbed-cli remove</span>
Adding a library from source control by URL is deprecated in Mbed CLI 2. Use `git rm SUBDIR` instead, and then remove the library's files. Various wrapper functions around `git` have been removed, to improve usability, maintainability, and to encourage direct use with more control.

<span id=dep>mbed-cli ls</span>
This feature is not supported in Mbed CLI 2. To view dependencies, use `mbed-tools deploy`.

### Tool configuration
<span id=dep>mbed-cli config</span>
Tool configuration has been removed from Mbed CLI 2. Compiler path is set up with CMake by adding to your `PATH`. Other configuration options are no longer supported:
- `target` must be explicitly passed on the command line when required. This avoids the current target being unclear due to global and local config files.
- `toolchain` must be explicitly passed on the command line when required, for the same reasons.
- `profile` should be passed explicitly when needed. This defaults to `develop`.
- `protocol` is no longer used in any subcommands.
- `depth` is no longer used in any subcommands. `git` should be used where custom depths are required.
- `cache` is no longer used in Mbed CLI 2.

<span id=dep>mbed-cli toolchain</span>
Setting a default toolchain is deprecated, as described above.

<span id=dep>mbed-cli target</span>
Setting a default target is deprecated, as described above.


### Creating an Mbed program
<span id=mod>mbed-tools new</span>
Creates a new Mbed program by default and supports only one command-line-option `--create-only`: create a program without fetching mbed-os.

<span id=dep>mbed new --program/--library</span>
By default a program will be created. To create a library, replace `add_executable()` with `add_library()` in your project `CMakeLists.txt`.

<span id=dep>mbed new --mbedlib</span>
Use of Mbed 2 is not supported with Mbed CLI 2.

<span id=dep>mbed new --scm</span>
The only source control management supported in Mbed CLI 2 is `git`, removing the need for an SCM option.

<span id=dep>mbed new --depth</span>
By default Mbed CLI 2 will use `depth=1` when fetching Mbed OS. To fetch more revisions, use `git` commands.

<span id=dep>mbed new --protocol</span>
Mbed CLI 2 will always use `https` as the protocol when fetching mbed-os. To use a different protocol, use `mbed-tools new --create-only` and then `git clone` mbed-os.

<span id=mod>mbed new --no-requirements</span>
In Mbed CLI 2, this is the standard behavior of `new`. Library dependencies are now checked out using the `deploy` subcommand.

<span id=dep>mbed new --offline</span>
Mbed CLI 2 does not cache dependencies.

### Importing an Mbed program

<span id=mod>mbed-tools import</span>
Clones an Mbed project and library dependencies, now with a depth of 1, rather than all revisions.

<span id=dep>mbed import --depth</span>
Mbed CLI 2 will by default use a depth of 1, fetching additional revisions only if needed for a specific commit. Specific depths can be cloned using `git` commands.

<span id=dep>mbed import --protocol</span>
Mbed CLI 2 supports standard `git` URL formats `protocol://host/directory` and `user@host:directory`, so this argument is not required.

<span id=dep>mbed import --insecure</span>
Mbed CLI 2 does not check URLs for standard port use, so this is no longer supported.

<span id=dep>mbed import --offline</span>
Mbed CLI 2 does not cache dependencies, so offline mode is not supported.

<span id=mod>mbed import --no-requirements</span>
This is now the standard behavior of `import`. Library dependencies are now checked out using the `deploy` subcommand.

### Configuring an Mbed program
<span id=new>mbed-tools configure</span>
This generates an Mbed OS CMake config file and writes it to `cmake_build/TARGET/PROFILE/TOOLCHAIN/` in the program directory. This can be used to build Mbed OS with standard `cmake` commands.

### Compiling an Mbed program

<span id=mod>mbed-tools compile</span>
Mbed CLI 2 builds to `cmake_build/TARGET/PROFILE/TOOLCHAIN/` rather than `BUILD/TARGET/TOOLCHAIN`. This allows building with multiple profiles, such as development and testing, in parallel.

<span id=mod>mbed compile --source</span>
Mbed CLI 2 will always use the current directory as the project directory, but Mbed OS directory can be passed by using `--mbed-os-path PATH`.

<span id=dep>mbed compile --library</span>
Replace `add_executable()` with `add_library()` in your project's `CMakeLists.txt` to compile the project as a library.

<span id=dep>mbed compile -D MACRO_NAME</span>
Use `mbed-tools configure` to generate the config file, and then from the build directory for your target and toolchain run `cmake --build . -D MACRO_NAME`. You can also use `target_compile_definitions` in your project's `CMakeLists.txt`.

<span id=dep>mbed compile -N ARTIFACT_NAME</span>
The compiled executable or library name is set to `APP_TARGET` as defined in the project's `CMakeLists.txt`

<span id=mod>mbed compile --build</span>
Mbed CLI 2 does not currently support selection of a build directory. Support is planned to be added later.

<span id=dep>mbed compile -S</span>
Board support must be checked using the `targets.json` file included with the version of Mbed OS in use. Mbed OS version support can be checked on os.mbed.com/platforms

### Testing an Mbed program
<span id=mod>mbed test</span>
Support is planned to be added at a later point.