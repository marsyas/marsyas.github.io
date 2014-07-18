---
---

# Building latest Marsyas on Mac OS X

These instructions are for building the latest development version of Marsyas.
It is assumed that you have already obtained the source code.
If not, please consult @ref{Get Marsyas sources} for instructions.


## Install prerequisits

- **CMake**

  Marsyas uses CMake to configure and guide its building process.

  [Download](http://www.cmake.org/cmake/resources/software.html) and install
  CMake. At some point you may be asked whether you want to install links to CMake
  to system locations - choose \"yes\": this will allow you to run CMake from the
  Terminal.

- **Xcode**

  Xcode development environment provides the Clang compiler required to compile
  Marsyas. If you want to use all Marsyas features, you need a compiler with
  a good C++11 support. Hence, please install the
  [latest Xcode](https://developer.apple.com/xcode/) version from Apple.

  Just as with CMake above, confirm installation of compilation tools to
  system locations, so as to make them available for use in the Terminal.



## Configure Marsyas using CMake

1. Start up the Terminal application.

2. Navigate to the top-level directory of Marsyas sources. For example:

        cd ~/marsyas

3. Create a build directory within, which will contain configuration
   options and compiled Marsyas libraries and programs. Navigate to the build
   directory:

        mkdir build
        cd build


3. Run CMake, passing it the Marsyas source directory as argument:

        cmake ..

CMake should now have generated a number of new files in the build directory,
including a file named `Makefile` which allows you to compile Marsyas.

If you would like to use CMake options to enable and disable specific features,
please see instructions in [Configuration with CMake](configure.html).


## Compile Marsyas using _make_

Still in the build directory, use the `make` command to compile Marsyas.

    make

*Geeky note: If your CPU has multiple cores (it is capable of running
multiple threads in parallel), you can shorten the compilation time by running
several instances of `make` in parallel by using the `-j` option followed by
the number of instances. The example below runs 3 parallel `make` instances:*

    make -j3

You could also compile Marsyas in Debug mode, which would help developers
discover bugs in case you run into troubles when using Marsyas. However,
Marsyas will run significantly slower when compiled in Debug mode.

To compile in Debug mode, you need to first use `cmake` to change a CMake
option named CMAKE_BUILD_TYPE, and then run `make`. Please mind the \".\"
at the end of the first command, to indicate the current directory:

    cmake -DCMAKE_BUILD_TYPE=Debug .
    make -j3


## Marsyas usage and system-wide installation

After compiling, you should have several Marsyas programs in the
`build/bin` subdirectory and the Marsyas library in the `build/lib`
subdirectory.

For example, assuming that you are in the `build` directory, you can run
the `sfplay` program to play an uncompressed sound file like this:

    ./bin/sfplay my_sound_file.wav

If you want to make the programs and the library available outside of your
build directory, you should use the following command to install all parts
of Marsyas to appropriate system locations:

    make install

By default, the above command will install programs and libraries under the
`/usr/local` prefix. You can change that by setting the CMake option
CMAKE_INSTALL_PREFIX to the desired prefix before installation:

    cmake -DCMAKE_INSTALL_PREFIX=~/marsyas-install .
    make install
