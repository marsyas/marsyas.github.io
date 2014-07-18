---
---

# Building latest Marsyas on Windows

These instructions are for building the latest development version of Marsyas.
It is assumed that you have already obtained the source code.
If not, please consult @ref{Get Marsyas sources} for instructions.

## Install prerequisits

- **CMake**

  Marsyas uses CMake to configure and guide its building process.

  [Download](http://www.cmake.org/cmake/resources/software.html) and run
  the CMake Windows installer. At some point you may be asked whether you want
  to make CMake available to the whole system - choose \"yes\": this will allow you
  to run CMake from Visual Studio Command Prompt.


- **Microsoft Visual Studio**

  We recommend to use the Microsoft Visual Studio compiler for C++ sources.

  [Download](http://www.microsoft.com/visualstudio/eng/downloads) and install
  Visual Studio. If you want to use all features of Marsyas, you will need
  at least Visual Studio 2013 or later (for C++11 support). The free \'Express\'
  version is sufficient.

## Choose correct Visual Studio Command Prompt

Visual Studio versions which support compilation of 64 bit programs provide you
with alternative versions of Command Prompt with the following labels and purposes:

- **x64** - for 64 bit compilation.
- **x86** - for 32 bit compilation.

Search the Start menu for the Command Prompt with one of the above labels,
according to your intended compilation mode.

*Geeky details: The different Command Prompt versions automatically
populate the PATH environment variable so as to provide appropriate versions
of compilation tools and libraries for the intended compilation mode.*


## Configure Marsyas using CMake

1. Start the appropriate Visual Studio Command Prompt, as described above.

2. Navigate to the top-level directory of Marsyas sources. Use the 'cd'
    command, passing it the path to Marsyas as argument. For example:

        cd C:\Users\John\marsyas

3. Create a build directory within, which will contain configuration
    options and compiled Marsyas libraries and programs. Navigate to the build
    directory:

        mkdir build
        cd build


4. Start CMake GUI, passing it the source directory (one level up: \"..\")
    as argument:

        cmake-gui ..


5. In CMake GUI: Click \"Configure\" to start auto-configuration of Marsyas
    for Windows.

6. In CMake GUI: A dialog will ask you to choose the desired \"generator\".
    From the drop-down menu, choose the appropriate one based on the
    Visual Studio version and compilation mode. Click \"Finish\".

    *Note:* \"Visual Studio 12\" corresponds to Visual Studio 2013. For
    64 bit compilation choose a generator which contains \"Win64\" in its name.

7. In CMake GUI: Click \"Generate\" to generate a Visual Studio solution for
    desired configuration. Exit CMake GUI.


CMake should now have generated a number of new files in the build directory,
including a Visual Studio solution file named \"marsyas.sln\".

If you would like to use CMake options to enable and disable specific features,
please see instructions in [Configuration with CMake](configure.html).


## Compile Marsyas using Visual Studio

Still in the build directory, use 'msbuild' (the Visual Studio build tool)
to compile Marsyas in Release mode:

    msbuild /p:Configuration=Release marsyas.sln

You could also compile Marsyas in Debug mode, which would help developers
discover bugs in case you run into troubles when using Marsyas. However,
Marsyas will run significantly slower when compiled in Debug mode:

    msbuild /p:Configuration=Debug marsyas.sln

After compiling, you should have Marsyas programs in the 'bin' subdirectory
and the Marsyas library in the 'lib' subdirectory.
