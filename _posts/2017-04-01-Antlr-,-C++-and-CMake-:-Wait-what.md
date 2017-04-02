---
layout: post
title: "Antlr4 for C++ with CMake: A practical example"
date: 2017-04-01
url: antlr4-cpp-cmake.html
---
Not long ago, I had to process a language's syntax for a rather big solo
project written in C++14 and using CMake 3.6 as the build system
(check it [here](https://github.com/blorente/naylang)). I had no idea about which tool to use at the time, and ANTLR sounded familiar. The last version (ANTLR 4) had a [freshly-made C++ target](http://www.soft-gems.net/index.php/tools/49-the-antlr4-c-target-is-here) merged into the main repo, which generated lexers and parsers in C++11, so I went ahead and spent the weekend integrating it in my project.

This is an account of the trials and conclusions I found.

## What's in the package?

The ANTLR 4 C++ target needs several things in order to work:

### The [ANTLR `.jar` file](www.antlr.org/download/antlr-4.7-complete.jar)

This program is the actual parser generator. It takes ANTLR grammar files as input (such as `MySuperAwesomeLanguage.g4`) and, when [run with the `-Dlanguage=Cpp` flag](https://github.com/antlr/antlr4/blob/master/doc/cpp-target.md), it creates parser and lexer classes:

```
$ antlr4 -Dlanguage=Cpp MyGrammar.g4
```
This will generate files such as `MyGrammarParser.h` and `MyGrammarLexer.h`. If so specified (e.g. with the flag -visitor), additional classes will be created.

As we will see later, we never actually have to invoke this command by hand.

### The ANTLR4CPP runtime

This is where the hard part comes. In order to use the newly generated classes, we need to compile and link them against the runtime, which is obtained from the [main repo](https://github.com/antlr/antlr4). Luckily, the C++ runtime also is built using CMake, which makes the integration at least 20% less painful.

The next sections illustrate the two easiest ways I found to integrate it into a project. They are not the only ones and they both rely on the `ExternalProject` CMake package, but after testing several other ways these ones were the easiest to me.

**NOTE: Under Linux, the runtime needs package `uuid-dev`. You can get it in Debian-based distros via `$ sudo apt-get install uuid-dev`**

## Compilation example

Beware: The compilation of the library takes a long time.

<script type="text/javascript" src="https://asciinema.org/a/4w7s0i71vtt2zki299lt603lo.js" id="asciicast-4w7s0i71vtt2zki299lt603lo" async></script>


## Basic: ExternalProject with remote

For those unaware, ExternalProject is a neat package from CMake that makes easy to include... well... projects from outside your project. That, combined with a handy file with the `.cmake` extension [included with the runtime](https://github.com/antlr/antlr4/blob/master/runtime/Cpp/cmake/ExternalAntlr4Cpp.cmake), makes it possible to add ANTLR 4 as a dependency with minimal changes to the project.

Let's take a moment to explore the contents of this magnificent file:

- A great comment introduction explaining what a minimal CMake project should look like to link against `antlr4cpp`.
- The `ExternalProject_Add` call to add `antlr4cpp` as an external dependency, by downloading it from GitHub and adding a compilation target. It also sets the useful variables `ANTLR4CPP_INCLUDE_DIR` (to include in your project's CMakeLists.txt) and `ANTLR4CPP_LIBS`, the directory where the compiled libraries will be stored.
- A handy macro that takes care of **generating the Lexer and Parser classes** and adding a compilation target `antlr4cpp_generation_<your_project_namespace>` and the handy variable `antlr4cpp_include_dirs_<your_project_namespace>` to include in your CMakeLists.txt.

With all this in hand, let's create a simple project that links against the library:

- Create a folder with the following structure, leaving `main.cpp` and `CMakeLists.txt` empty. You can get example grammars from  [here](https://github.com/antlr/antlr4/tree/master/runtime/Cpp/demo).
```
test_antlr/
|-- cmake/
|---- ExternalAntlr4Cpp.cmake
|-- thirdparty/
|---- antlr-4.7-complete.jar
|-- main.cpp
|-- CMakeLists.txt
|-- grammar/
|---- TLexer.g4
|---- TParser.g4
```

- Create `main.cpp`. The aim is just to compile agains the runtime and to be able to include the generated Parser and Lexer, so it should be pretty simple:

  ```c++
  #include <iostream>
  #include <antlr4-runtime.h>
  #include "TParser.h"

  int main() {
    std::cout << "Hello World" << std::endl;
    return 0;
  }
  ```
- Create CMakeLists.txt. This is where the meat of the potato begins, if you get my meaning.

  - First, let's define some standard CMake targets, as though the library didn't exist.

    ```cmake
    # CMakeLists.txt

    # minimum required CMAKE version
    CMAKE_MINIMUM_REQUIRED(VERSION 3.5)

    # compiler must be 11 or 14
    SET (CMAKE_CXX_STANDARD 14)

    add_executable(test_antlr main.cpp)
    ```
  - Then, we have to include the package and make it discoverable by our project.

    ```cmake
    # CMakeLists.txt

    CMAKE_MINIMUM_REQUIRED(VERSION 3.5)

    LIST( APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake )
    # ...
    SET (CMAKE_CXX_STANDARD 14)

    # add external build for antlrcpp
    include( ExternalAntlr4Cpp )
    # ...
    ```

  - As a small detail, we have to tell the package where the `.jar` file is located.

    ```cmake
    # CMakeLists.txt

    # set variable pointing to the antlr tool that supports C++
    set(ANTLR4CPP_JAR_LOCATION ${PROJECT_SOURCE_DIR}/thirdparty/antlr/antlr-4.7-complete.jar)
    ```

  - We can take advantage of those handy variables defined by the package:

    ```cmake
    # CMakeLists.txt

    # Include the runtime to compile against
    include_directories( ${ANTLR4CPP_INCLUDE_DIRS} )
    link_directories( ${ANTLR4CPP_LIBS} )
    message(STATUS "Found antlr4cpp libs: ${ANTLR4CPP_LIBS} and includes: ${ANTLR4CPP_INCLUDE_DIRS} ")

    # Call macro to add lexer and grammar to your build dependencies.
    # NOTE: Here, we define "antlrcpptest" as our project's namespace
    antlr4cpp_process_grammar(demo antlrcpptest
      ${CMAKE_CURRENT_SOURCE_DIR}/TLexer.g4
      ${CMAKE_CURRENT_SOURCE_DIR}/TParser.g4)
    # include generated files in project environment
    include_directories(${antlr4cpp_include_dirs_antlrcpptest})
    ```

  - And lastly, we have to change the compilation line to link against the libraries:

    ```cmake
    # CMakeLists.txt

    # add generated grammar to demo binary target
  add_executable(test_antlr main.cpp ${antlr4cpp_src_files_antlrcpptest})
  add_dependencies(test_antlr antlr4cpp antlr4cpp_generation_antlrcpptest)
  target_link_libraries(test_antlr antlr4-runtime)
    ```

  - Presto! When we try to build our project, it will first download a fresh copy from the repo or pull from it to get the latest changes. Then, it will build the library and generate the parser classes, before building and linking to your project. The complete `CMakeLists.txt` file can be found [here](https://github.com/blorente/antlr-4.7-cpp-cmake-base/blob/master/CMakeLists.txt) (with a slightly outdated version of the `.jar`).

## Optional: ExternalProject with local copy

Now I personally don't like this method, because it already takes long enough to build the runtime, and this would have to download it every time it is freshly built. That can amount to tens of minutes of delay, which when combined with Travis-Ci' slow build times can make integration a real pain.

One way to alleviate this is to distribute a frozen copy of the runtime along with your project (for example, in a zip file), which would then be unpacked and built locally instead of downloading it. In addition to considerably shortening build times, this has the advantage that we work with a stable version of the library, and it will not change from install to install (something surprisingly difficult these days).

Here are the steps to get that working:

- Download a copy of the [ANTLR 4 repo](https://github.com/antlr/anlr4).

- Place it wherever you'd like. I placed it alongside the `.jar`:

  ```
  test_antlr/
  |-- thirdparty/
  |---- antlr-4.7-complete.jar
  |---- antlr4-master.zip
  ```

- Change the `ExternalAntlr4Cpp.cmake` file to include your zip as an URL instead of the Git repository:

  ```cmake
  # ExternalAntlr4Cpp.cmake

  # Add definitions for the local repository
  set(ANTLR4CPP_LOCAL_ROOT ${CMAKE_BINARY_DIR}/locals/antlr4cpp)
  SET(ANTLR4CPP_LOCAL_REPO ${PROJECT_SOURCE_DIR}/thirdparty/antlr/antlr4-master.zip)

  # Make the following changes to the _Add rule
  ExternalProject_ADD(
    #--External-project-name------
    antlr4cpp
    # ...
    #--Core-directories-----------
    PREFIX             ${ANTLR4CPP_LOCAL_ROOT}
    #--Download step--------------
    URL                 ${ANTLR4CPP_LOCAL_REPO}
    # Comment these out
    # GIT_REPOSITORY     ${ANTLR4CPP_EXTERNAL_REPO}
    # GIT_TAG          ${ANTLR4CPP_EXTERNAL_TAG}
    # ...
    # And this
    # UPDATE_COMMAND     ${GIT_EXECUTABLE} pull
    # ...
    # INSTALL_COMMAND    ""
  )
  ```

And that's it! Simple enough right? We didn't even have to touch `CMakeLists.txt`!

## Just for funzies: Travis-CI integration

If your project uses Travis as a CI service, you might want to know how these changes affect to your `.travis.yml` file. Well, as a small bonus, here's how to configure Travis to build and run your project with ANTLR:

- Get the basic stuff, to compile with `gcc`:

  ```yml
  language: cpp
  compiler:
  - gcc
  ```

- Update package repositories to get the latest version of `gcc`:

  ```yml
  before_install:
  - sudo add-apt-repository ppa:ubuntu-toolchain-r/test -y
  - sudo apt-get update
  ```

- Create a deps folder to store the depenencies:

  ```yml
  - DEPS_DIR="${TRAVIS_BUILD_DIR}/deps"
  - mkdir -p ${DEPS_DIR} && cd ${DEPS_DIR}
  ```

- Obtain the latest copies of CMake and `gcc`:

  ```yml
  - |
    if [[ "${TRAVIS_OS_NAME}" == "linux" ]]; then
      CMAKE_URL="https://cmake.org/files/v3.7/cmake-3.7.2-Linux-x86_64.tar.gz"
      mkdir -p cmake && travis_retry wget --no-check-certificate --quiet -O - ${CMAKE_URL} | tar --strip-components=1 -xz -C cmake
      export PATH=${DEPS_DIR}/cmake/bin:${PATH}
    else
      brew upgrade cmake || brew install cmake
    fi
  - cmake --version
  - if [ "$CXX" = "g++" ]; then sudo apt-get install -qq g++-6; fi
  - if [ "$CXX" = "g++" ]; then export CXX="g++-6" CC="gcc-6"; fi
  ```

- Install uuid (required by ANTLR under Linux)

  ```yml
  - sudo apt-get install -y uuid-dev
  ```

- And then the regular out-of-source CMake build:

  ```yml
  script:
  - cd ${TRAVIS_BUILD_DIR}
  - mkdir build
  - cd build
  - cmake -G "Unix Makefiles" ..
  - make -j2 VERBOSE=1
  - ./test_antlr
  ```

And that's it! You are now ready to integrate those beautiful grammars into your C++11 projects!

Please send any feedback to [blorente@ucm.es](mailto:blorente@ucm.es), and I'll try to answer as fast as possible.

## [GET THE FULL SOURCE CODE](https://github.com/blorente/antlr-4.7-cpp-cmake-base)
