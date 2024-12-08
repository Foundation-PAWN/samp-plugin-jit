[JIT plugin][github]
====================

[![Version][version_badge]][version]
[![Build Status][build_status]][build]
[![Build Status - Windows][build_status_win]][build_win]

This plugin is an implementation of a just-in-time (JIT) compiler for Pawn.
It compiles AMX bytecode into native x86 code on the fly and hooks into the
AMX API called by the SA-MP server to replace the VM's interpreter.

Although this is a SA-MP plugin the code is organized into a library called
amxjit which theoretically could be reused in other host programs, although 
I don't know of such applications.

Installation
------------

1. Download a binary package form the [Releases][download] page on Github 
   or build it yourself from source code (see
   [Building from source code](#building-from-source-code)).
2. Extract/copy `jit.so` or `jit.dll` to `<sever>/plugins/`.
3. Add `jit` (Windows) or `jit.so` (Linux) to the `plugins` line of your 
   server.cfg.

Binary archives come with an include file (`jit.inc`) that contains
some helper functions that you may find useful. But **you don't need to
include** it to be able to use the plugin, it's not required.

Usage
-----

Apart from installing the plugin you don't have to configure anything else.
If it gets loaded and you don't see any errors then it means that you code 
has been JIT-ed successfully.

If you see an error about an invalid instructions, check if you are using
YSI or other advanced libraries that use `#emit` (their code may be
incompatible with JIT). If using YSI try to update to the latest version
first. If that doesn't help you are out of luck, sorry. See
[Limitations](#limitations).

Keep in mind that using JIT usually helps with computation-intenstive tasks,
i.e. CPU bound code. It will not improve performance of those parts of code
that spend time on I/O operations such as reading from a database or the 
file system.

Building from source code
-------------------------

If you want to build JIT from source code, e.g. to fix a bug and submit a 
pull request, simply follow the steps below. You will need a C++ compiler
and CMake.

### Linux

Install gcc and g++, make and cmake. On Ubuntu you would do that like so:

```
sudo apt-get install gcc g++ make cmake
```

If you're on a 64-bit system you'll need additional packages for compiling
for 32-bit:

```
sudo apt-get install gcc-multilib g++-multilib
```

For CentOS:

```
yum install gcc gcc-c++ cmake28 make
```

Now you're ready to build the plugin:

```
cd jit
mkdir build && cd build
cmake ../ -DCMAKE_BUILD_TYPE=Release -DBUILD_TESTING=OFF
make
```

### Windows

You'll need to install CMake and Visual Studio (Express edition will suffice).
After that, either run cmake from the command line:

```
cd jit
mkdir build && cd build
path/to/cmake.exe ../ -DBUILD_TESTING=OFF
```

or do the same from cmake-gui. This will generate a Visual Studio project in
the build folder.

To build the project:

```
path/to/cmake.exe --build . --config Release
```

You can also build it from within Visual Studio: open build/jit.sln
and go to menu -> Build -> Build Solution (or just press F7).

Limitations
-----------

Anything that doesn't use `#emit` will probably work perfectly fine. 
Otherwise, it depends on the code. There are some advanced `#emit` hacks 
that simply won't work with this JIT. Self-modifying code is one example 
(although in some caes it's possible to fix, see 
[Detecting JIT at runtime][wiki-detecting]).

If you're using [YSI][ysi] this plugin most likely will not work for you and
will simply crash your server. However, with the recent versions of YSI this
may be no longer true. Just try it for yourself and see if it works.

Plugins that hook `amx_Exec()`, for example [CrashDetect][crashdetect], can't 
be used together with this plugin. If such hook is detected you will get an 
error at startup and your code will run through the default interpreter as 
usual.

How it works
------------

This is a pretty simple JIT. Code is compiled once and stored in memory until
the corresponding AMX is unloaded. Because of this there is a small delay
during script loading/initialization.

Majority of the compilation process is a direct translation of AMX opcodes 
into sequences of corresponding machine instructions using essentially a giant
`switch` statement. But there are some places where the JIT compiler tries to 
be a little bit smarter: for instance, it will replace calls to common 
floating-point functions (those found in float.inc) with equivalent FP
instructions built into the CPU to avoid the overhead of native function calls.

Code generation is done via [AsmJit][asmjit], a great assembler library for 
x86/x86-64 :thumbsup:. There also was an attempt to use LLVM as an alternative 
backend but it was abandoned...

License
-------

Licensed under the 2-clause BSD license. See [LICENSE.txt](LICENSE.txt).

[github]: https://github.com/Zeex/samp-plugin-jit
[version]: http://badge.fury.io/gh/Zeex%2Fsamp-plugin-jit
[version_badge]: https://badge.fury.io/gh/Zeex%2Fsamp-plugin-jit.svg
[build]: https://travis-ci.org/Zeex/samp-plugin-jit
[build_status]: https://travis-ci.org/Zeex/samp-plugin-jit.svg?branch=master
[build_win]: https://ci.appveyor.com/project/Zeex/samp-plugin-jit/branch/master
[build_status_win]: https://ci.appveyor.com/api/projects/status/v1duwc12h7vq4vvu/branch/master?svg=true
[download]: https://github.com/Zeex/samp-plugin-jit/releases
[asmjit]: https://github.com/kobalicek/asmjit
[wiki-detecting]: https://github.com/Zeex/samp-plugin-jit/wiki/Detecting-JIT-at-runtime
[ysi]: https://github.com/Y-Less/YSI
[crashdetect]: https://github.com/Zeex/samp-plugin-crashdetect
