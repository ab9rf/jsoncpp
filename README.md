Introduction
------------

[JSON][json-org] is a lightweight data-interchange format. It can represent
numbers, strings, ordered sequences of values, and collections of name/value
pairs.

[json-org]: http://json.org/

JsonCpp is a C++ library that allows manipulating JSON values, including
serialization and deserialization to and from strings. It can also preserve
existing comment in unserialization/serialization steps, making it a convenient
format to store user input files.


Using JsonCpp in your project
-----------------------------

The recommended approach to integrating JsonCpp in your project is to build the
the amalgamated source (a single `.cpp` file) with your own build system. This
ensures consistency of compilation flags and ABI compatibility. See the section
"Generating amalgamated source and header" for instructions.
  
The `include/` should be added to your compiler include path. Jsoncpp headers
should be included as follow:

    #include <json/json.h>

If JsonCpp was build as a dynamic library on Windows, then your project needs to
define the macro `JSON_DLL`.


Building and testing with new CMake
-----------------------------------

[CMake][] is a C++ Makefiles/Solution generator. It is usually available on most
Linux system as package. On Ubuntu:

    sudo apt-get install cmake

[CMake]: http://www.cmake.org

Note that Python is also required to run the JSON reader/writer tests. If
missing, the build will skip running those tests.

When running CMake, a few parameters are required:

* a build directory where the makefiles/solution are generated. It is also used
  to store objects, libraries and executables files.
* the generator to use: makefiles or Visual Studio solution? What version or
  Visual Studio, 32 or 64 bits solution? 

Steps for generating solution/makefiles using `cmake-gui`:

* Make "source code" point to the source directory.
* Make "where to build the binary" point to the directory to use for the build.
* Click on the "Grouped" check box.
* Review JsonCpp build options (tick `JSONCPP_LIB_BUILD_SHARED` to build as a
  dynamic library).
* Click the configure button at the bottom, then the generate button.
* The generated solution/makefiles can be found in the binary directory.

Alternatively, from the command-line on Unix in the source directory:

    mkdir -p ../build/debug
    cd ../build/debug
    cmake -DCMAKE_BUILD_TYPE=debug -DJSONCPP_LIB_BUILD_SHARED=OFF -G "Unix Makefiles" ../../jsoncpp-src
    make

Running `cmake -`" will display the list of available generators (passed using
the `-G` option).

By default CMake hides compilation commands. This can be modified by specifying
`-DCMAKE_VERBOSE_MAKEFILE=true` when generating makefiles.


Building and testing with SCons
-------------------------------

**Note:** The SCons-based build system is deprecated. Please use CMake; see the
section above.

JsonCpp can use [Scons][] as a build system. Note that SCons requires python to
be installed.

[SCons]: http://www.scons.org/

Invoke SCons as follows:

    scons platform=$PLATFORM [TARGET]

where `$PLATFORM` may be one of:

* `suncc`: Sun C++ (Solaris)
* `vacpp`: Visual Age C++ (AIX)
* `mingw`
* `msvc6`: Microsoft Visual Studio 6 service pack 5-6
* `msvc70`: Microsoft Visual Studio 2002
* `msvc71`: Microsoft Visual Studio 2003
* `msvc80`: Microsoft Visual Studio 2005
* `msvc90`: Microsoft Visual Studio 2008
* `linux-gcc`: Gnu C++ (linux, also reported to work for Mac OS X)

If you are building with Microsoft Visual Studio 2008, you need to set up the
environment by running `vcvars32.bat` (e.g. MSVC 2008 command prompt) before
running SCons.


Running the tests manually
--------------------------

Note that test can be run using SCons using the `check` target:

    scons platform=$PLATFORM check

You need to run tests manually only if you are troubleshooting an issue.

In the instructions below, replace `path/to/jsontest` with the path of the
`jsontest` executable that was compiled on your platform.

    cd test
    # This will run the Reader/Writer tests
    python runjsontests.py path/to/jsontest
    
    # This will run the Reader/Writer tests, using JSONChecker test suite
    # (http://www.json.org/JSON_checker/).
    # Notes: not all tests pass: JsonCpp is too lenient (for example,
    # it allows an integer to start with '0'). The goal is to improve
    # strict mode parsing to get all tests to pass.
    python runjsontests.py --with-json-checker path/to/jsontest
    
    # This will run the unit tests (mostly Value)
    python rununittests.py path/to/test_lib_json
    
    # You can run the tests using valgrind:
    python rununittests.py --valgrind path/to/test_lib_json


Building the documentation
--------------------------

Run the python script doxybuild.py from the top directory:

python doxybuild.py --open --with-dot

See doxybuild.py --help for options. 

Notes that the documentation is also available for download as a tarball. 
The documentation of the latest release is available online at:
http://jsoncpp.sourceforge.net/

Generating amalgamated source and header
----------------------------------------

JsonCpp is provided with a script to generate a single header and a single
source file to ease inclusion in an existing project.

The amalgamated source can be generated at any time by running the following
command from the top-directory (requires python 2.6):

python amalgamate.py

It is possible to specify header name. See -h options for detail. By default,
the following files are generated:
- dist/jsoncpp.cpp: source file that need to be added to your project
- dist/json/json.h: header file corresponding to use in your project. It is
equivalent to including json/json.h in non-amalgamated source. This header
only depends on standard headers. 
- dist/json/json-forwards.h: header the provides forward declaration
of all JsonCpp types. This typically what should be included in headers to
speed-up compilation.

The amalgamated sources are generated by concatenating JsonCpp source in the
correct order and defining macro JSON_IS_AMALGAMATION to prevent inclusion
of other headers.

Adding a reader/writer test
---------------------------

To add a test, you need to create two files in test/data:
- a TESTNAME.json file, that contains the input document in JSON format.
- a TESTNAME.expected file, that contains a flatened representation of 
  the input document.
  
TESTNAME.expected file format:
- each line represents a JSON element of the element tree represented 
  by the input document.
- each line has two parts: the path to access the element separated from
  the element value by '='. Array and object values are always empty 
  (e.g. represented by either [] or {}).
- element path: '.' represented the root element, and is used to separate 
  object members. [N] is used to specify the value of an array element
  at index N.
See test_complex_01.json and test_complex_01.expected to better understand
element path.


Understanding reader/writer test output
---------------------------------------

When a test is run, output files are generated aside the input test files. 
Below is a short description of the content of each file:

- test_complex_01.json: input JSON document
- test_complex_01.expected: flattened JSON element tree used to check if 
    parsing was corrected.

- test_complex_01.actual: flattened JSON element tree produced by 
    jsontest.exe from reading test_complex_01.json
- test_complex_01.rewrite: JSON document written by jsontest.exe using the
    Json::Value parsed from test_complex_01.json and serialized using
    Json::StyledWritter.
- test_complex_01.actual-rewrite: flattened JSON element tree produced by 
    jsontest.exe from reading test_complex_01.rewrite.
test_complex_01.process-output: jsontest.exe output, typically useful to
    understand parsing error.

License
-------
  
See file LICENSE for details. Basically JsonCpp is licensed under 
MIT license, or public domain if desired and recognized in your jurisdiction.

