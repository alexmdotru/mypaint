# Windows binary build scripting

This directory contains a script for making binary builds, and runtime
support scripts for the Windows platform.

## Dependencies

Before making a Windows build with these scripts, you must first install
MSYS2, a toolchain consisting of open source software. Follow the
instructions at <https://msys2.github.io/>, and be sure to update it
before trying the commands below.

Inno Setup is supported, for creating "setup.exe" installers. Download
v5.5 or above from <http://www.jrsoftware.org/>, and install it. You may
need to add the directory containing Inno Setup's command line installer
`ISCC.exe` to your path if you install to a non-standard location.

All the build-time and runtime dependencies will be installed for you by
the build script. Making these builds can be a lengthy task, needs a Gb
or two of disk space, and some network bandwidth (github, and
sourceforge for MSYS2). On the upside however, lots of the downloaded
stuff will be cached for subsequent runs.

We're hoping to make this part of the releases starting with the 1st
beta release for 1.2.0, so if you get any success or failure, please
[feed it back to us on Twitter][tweetfb].

## Running

The main [`build.sh`](build.sh) script must be run from one of the
MINGW32 or MINGW64 shells which ship with MSYS. It will initially
populate this environment with the tools needed for the build, and keep
them up to date on subsequent runs.

To get started, start either "MinGW-w64 Win32 Shell" for MINGW32, or its
"Win64" buddy if you want a MINGW64 build. Then run the following
commands (with whatever modifications or omissions you need) in that
shell to fetch MyPaint from git:

    $ cd /g/my/devel
    $ pacman -S git
    $ git clone https://github.com/mypaint/mypaint.git
    $ cd mypaint

This assumes that you want MyPaint's source in `G:\my\devel\mypaint`.
The rest of the setup is automated:

    $ windows/build.sh --show-output

This should be enough to build MyPaint! The `build.sh` script performs
the following steps:

* Updates your MSYS2's MSYS/MINGW32/MINGW64 environments with everything
  needed for the build.
* Force-updates MyPaint's submodules to be at the expected version.
* Exports MyPaint from git using `../release.sh`,
  running all the tests as it does so.
  This means that your pending changes must be committed first.
* Creates a target environment, and populates all of MyPaint's runtime
  dependencies. These packages are deployed from MSYS2's cache, so
  network impact is kept minimal.
* Installs MyPaint into the target.
* Cleans unneccessary files from the target.
* Zips up the target folder in the form of a standalone zipfile.
* Runs Inno Setup's command line "setup.exe" compiler to make an
  installer distribution.
* Shows the output folder in Windows, if you told it to.

Run `windows/build.sh --help` to see options.

## Outputs

The build script's final outputs are written into a temporary folder in
the form of large zipfiles and .exe files.

If the `--show-output` option is specified, the temporary folder is then
opened in Windows Explorer with `start` after a successful build so that
you can copy the files somewhere else.

### Standalone build

This output is essentially a zipped folder
which, when unpacked, contains everything needed to run MyPaint.

Standalone builds work as they are, but they are quite
developer-focused.  Their `mypaint.cmd` launch script flashes up a
terminal briefly before launching MyPaint, and it doesn't have a nice
icon ☺ . The launch script also expects to be able to write cache files
into the `mingw32` or `mingw64` subfolder each time you run it.

### Installer build

The build script creates an Inno Setup compiler script, `mypaint.iss`,
and then tries to run it with `ISCC`. It's based on the template file
named [`mypaint.iss.in`](mypaint.iss.in) in this directory.

Our build script assumes that you installed Inno Setup
in `C:\Program Files (x86)\Inno Setup 5`,
but if your system is different, you will need to add the correct folder
to your system's `%PATH%` before running MyPaint's `build.sh`.

If `build.sh` can't run `ISCC`, you can still create a `setup.exe` by
double clicking on the generated `.iss` file.

[tweetfb]: https://twitter.com/intent/tweet?text=@MyPaintApp%20I%20tried%20windows/build.sh%20on%20Win%20%3CVERSION%3E,%20and...