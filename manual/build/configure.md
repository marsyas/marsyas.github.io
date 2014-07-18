---
---

# Configuration with CMake

Marsyas uses CMake to configure and guide its building process. CMake allows
for automatic configuration according to the current platform, as well as
user-configurable building options. It allows you to enable and disable
specific Marsyas features.

After configuration, CMake creates files that contain instructions for the
next step of the building process. This may be:

- Makefiles on Linux, Mac OS X or MinGW
- Xcode projects on Mac OS X
- MS Visual Studio solutions on Windows.


## Running CMake

**Note:** The following instructions assume that you have done the
prerequisite steps described in @ref{Step-by-step building instructions}:

1. Obtained Marsyas sources
2. Installed CMake and other prerequisite software
3. Created a `build` directory within the top-level Marsyas
   directory and navigated your command-line to the `build` directory

There are several ways that configuration using CMake can be performed.

### Using CMake GUI

**Note:** On Linux, you will need to install an extra package for the
CMake GUI. On Debian/Ubuntu the package name is `cmake-qt-gui`.

You can start CMake GUI either graphically via the list of
applications installed on your system, or by using the `cmake-gui` command
in the Linux console, OS X Terminal, or Windows Command Prompt.

- If you start it the graphical way, you will need to set the Marsyas
  source and build directories manually in the CMake GUI

- If you start it using the 'cmake-gui' command, you can pass it the
  Marsyas source directory as argument, and the current directory will
  be used as the build directory. For example, assuming that you are in the
  `build` directory:

      cmake-gui ..


CMake GUI contains a list of available options: the left column lists option
names, and the right column their editable values. By moving the mouse cursor
over an option name, its description will pop up. To perform configuration,
follow the steps below:

1. If Marsyas has not been configured before, the list of options will be
   empty. Click "Configure" to do the initial auto-configuration.

2. Modify options as desired, and click "Configure" again.

3. Once you are satisfied with options, click "Generate" to apply them.

4. You can now exit CMake GUI and proceed with the compilation step of
   the building process (see @ref{Step-by-step building instructions}).


### Entirely on command-line

You can also perform CMake configuration entirely on command-line.
Options are controlled by using the `cmake` command and passing it arguments
of the form `-D<option name>=<value>`. Values can be paths to
directories and files, or simply `ON` and `OFF` (or equivalently,
`TRUE` and `FALSE`). Multiple options can be changed using the same command.

For example, assuming that you are in the `build` directory:

    cmake -DMARSYAS_AUDIOIO=OFF -DMARSYAS_TESTS=ON ..

After invoking the `cmake` command one or multiple times to apply desired
options, you can proceed with the compilation step of the building process
(see @ref{Step-by-step building instructions})


### More alternatives

There are more alternative ways to use CMake. Please read detailed instructions
on the CMake website:

- [General instructions](http://www.cmake.org/cmake/help/runningcmake.html)
- [Command-line usage](http://www.cmake.org/cmake/help/v2.8.11/cmake.html#section_Usage)


## Most prominent options

The following is a list of most important and commonly used options provided
either by Marsyas or CMake itself.


### Input / output

- MARSYAS_AUDIOIO

  This enables audio input/output.  Requires DirectX on Windows and
  either JACK, ALSA or OSS on Linux.  MacOS X audio support is built-in
  with the basic developer tools.

  **Note:** This option requires C++11 support - in other words,
  the WITH_CPP11 option must also be enabled (described below).

- MARSYAS_MIDIIO

  This enables midi input/output.  Requires DirectX on Windows and
  either ALSA or OSS on Linux.  MacOS X audio support is built-in
  with the basic developer tools.

**Note:** Audio and MIDI IO support also depend on WITH_JACK, WITH_ALSA,
and WITH_OSS options, described below.


### Optional software

All of these options require additional software to be
**installed and properly configured**.

- WITH_CPP11

  Enables compilation in C++11 mode. If disabled, Marsyas will be compiled with
  limited functionality. Specifically, the audio IO and multi-threading support
  require this option to be enabled.

  This option requires a compiler with adequate C++11 support.
  Minimum required compiler versions are ensured by CMake, and reported if not
  satisfied.

- WITH_MAD

  mp3 audio decoding with
  @uref{http://sourceforge.net/projects/mad/, LibMAD}

- WITH_VORBIS

  ogg vorbis audio decoding with libvorbis - it requires

- WITH_GSTREAMER

  Use GStreamer as an audio source

- WITH_QT

  Builds the Qt5 GUI applications. Most Marsyas GUI applications are of this type.
  Requires Qt 5.0 or higher.

- WITH_QT4

  Builds the Qt4 GUI applications. There are only a few unmaintained Marsyas GUI
  applications of this type, preserved mostly for inspiration.
  Requires Qt 4.2.3 or higher.

- WITH_SWIG

  Builds SWIG bindings.  This option enables the following
  sub-options: WITH_SWIG_PYTHON, WITH_SWIG_JAVA, WITH_SWIG_LUA, and
  WITH_SWIG_RUBY.


  - WITH_SWIG_PYTHON

    Use Swig to generate Python bindings

  - WITH_SWIG_JAVA

    Use Swig to generate Java bindings

  - WITH_SWIG_LUA

    Use Swig to generate Lua bindings

  - WITH_SWIG_RUBY

    Use Swig to generate Ruby bindings

- WITH_MATLAB

  Builds the MATLAB engine interface.

- WITH_VAMP

  Build plugins for Vamp (see @ref{SonicVisualiser Vamp Plugins} for more information).



Linux-specific:


- WITH_JACK

  Enables audio IO using JACK, if available.

- WITH_ALSA

  Enables audio and MIDI IO using ALSA, if available.

- WITH_OSS

  Enables audio and MIDI IO using OSS, if available.


### Code messages and optional portions

- MARSYAS_ASSERT

  Turns on assertions.

- MARSYAS_PROFILING

  Turns on profiling.

- MARSYAS_DEBUG

  Turns on debugging info (large performance penalty).

- DISTRIBUTED

  (_advanced_ option) experimental code for distributed
  systems.


### Message logging

These are _advanced_ options.

- MARSYAS_LOG_WARNINGS

- MARSYAS_LOG_DEBUGS

- MARSYAS_LOG_DIAGNOSTICS

- MARSYAS_LOG2FILE

- MARSYAS_LOG2STDOUT

- MARSYAS_LOG2GUI


### Testing

- MARSYAS_TESTS

  Build Marsyas tests, so they can be run using @code{make test}.


### Documentation

- MARSYAS_DOCUMENTATION_ONLY

  If enabled, only build documentation, not program sources. This allows to
  build documentation without even the presence of a compiler or any requirements
  related to program code.
