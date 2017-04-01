---
layout: post
title: "Antlr4 for C++ with CMake: A practical example"
date: 2017-04-01
url: antlr4-cpp-cmake.html
---
Not long ago, I had to process a language's syntax for a rather big solo
project written in C++14 and using CMake 3.6 as the build system
(check it [here](https://github.com/blorente/naylang)). At the time I had no idea about which tool to use and ANTLR sounded familiar. The las version (ANTLR 4) had a [freshly-made C++ target](http://www.soft-gems.net/index.php/tools/49-the-antlr4-c-target-is-here) merged into the main repo, which generated lexers and parsers in C++11, so I went ahead and spent the weekend integrating it in my project.

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

## Option 1: ExternalProject with remote



## Option 2: ExternalProject with local copy
