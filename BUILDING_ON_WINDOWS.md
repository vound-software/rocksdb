Orginal post of instruction below is at : https://github.com/EighteenZi/rocksdb_wiki/blob/master/Building-on-Windows.md
==========================================================================================================================

Building on Windows
This is a simple step-by-step explanation of how I was able to build RocksDB (or RocksJava) and all of the 3rd-party libraries on Microsoft Windows 10. The Windows build system was already in place, however it took some trial-and-error for me to be able to build the 3rd-party libraries and incorporate them into the build.

Pre-requisites
Microsoft Visual Studio 2015 (Community)
CMake
Git - I used the Windows Git Bash.
wget
Steps
Create a directory somewhere on your machine that will be used a container for both the RocksDB source code and that of its 3rd-party dependencies. On my machine I used d:\code\3.sdks.and.libs\vound-code, from hereon in I will just refer to it as %CODE_HOME%; which can be set as an environment variable, i.e. SET CODE_HOME=d:\code\3.sdks.and.libs\vound-code

Build GFlags
======================================
1. open visual studio with no code (in my case it was 2022)
2. run "tools/command line/developer" command prompt menu option -- this simplifies things and opens right env
3. following code execute in opened comamnd prompt

cd %CODE_HOME%
wget https://github.com/gflags/gflags/archive/v2.2.0.zip
unzip v2.2.0.zip
cd gflags-2.2.0
mkdir target
cd target
cmake ..

Open the project in Visual Studio, create a new x64 Platform by copying the Win32 platform and selecting x64 CPU. Close Visual Studio.
change runtime library to MT , and build from visual studio -- record location of generated static libs to be used to modify thirdparty.inc


Build Snappy
===============================
1. open visual studio with no code (in my case it was 2022)
2. run "tools/command line/developer" command prompt menu option -- this simplifies things and opens right env
3. following code execute in opened comamnd prompt

cd %CODE_HOME%
wget https://github.com/google/snappy/releases/tag/1.1.9
unxzip 1.1.9
cd snappy-1.1.9

edit CMakeLists.txt changing following settings to : 
option(SNAPPY_BUILD_TESTS "Build Snappy's own tests." OFF)
option(SNAPPY_BUILD_BENCHMARKS "Build Snappy's benchmarks" OFF)
save changes 

mkdir build
cd build
cmake -S .. -B .
devenv snappy.sln /upgrade

open generated build/snappy.sln in visual studio change runtime library to MT , and build from visual studio -- record location of generated static libs to be used to modify thirdparty.inc

Build LZ4
=========================================
1. open visual studio with no code (in my case it was 2022)
2. run "tools/command line/developer" command prompt menu option -- this simplifies things and opens right env
3. following code execute in opened comamnd prompt

cd %CODE_HOME%
wget https://github.com/lz4/lz4/archive/v1.7.5.zip
unzip v1.7.5.zip
cd lz4-1.7.5
cd visual\VS2010
devenv lz4.sln /upgrade
open solution in visual studio change runtime library to MT , and build from visual studio -- record location of generated static libs to be used to modify thirdparty.inc


Build ZLib
=========================================================
1. open visual studio with no code (in my case it was 2022)
2. run "tools/command line/developer" command prompt menu option -- this simplifies things and opens right env
3. following code execute in opened comamnd prompt

cd %CODE_HOME%
wget http://zlib.net/zlib1211.zip
unzip zlib1211.zip
cd zlib-1.2.11\contrib\vstudio\vc14
Edit the file zlibvc.vcxproj, changing <command>cd ..\..\contrib\masmx64 bld_ml64.bat</command> to <command>cd ..\..\masmx64 bld_ml64.bat</command>.

msbuild zlibvc.sln /p:Configuration=Debug /p:Platform=x64
msbuild zlibvc.sln /p:Configuration=Release /p:Platform=x64

copy x64\ZlibDllDebug\zlibwapi.lib x64\ZlibStatDebug\
copy x64\ZlibDllRelease\zlibwapi.lib x64\ZlibStatRelease\


Build RocksDB
=========================================================
1. open visual studio with no code (in my case it was 2022)
2. run "tools/command line/developer" command prompt menu option -- this simplifies things and opens right env
3. following code execute in opened comamnd prompt
4. rocksdb folder contains rocksdb cloned from our github

cd %CODE_HOME%
cd rocksdb 
Edit the file %CODE_HOME%\rocksdb\thirdparty.inc to have these changes:

set(GFLAGS_HOME $ENV{THIRDPARTY_HOME}/gflags-2.2.0)
set(GFLAGS_INCLUDE ${GFLAGS_HOME}/target/include)
set(GFLAGS_LIB_DEBUG ${GFLAGS_HOME}/target/lib/Debug/gflags_static.lib)
set(GFLAGS_LIB_RELEASE ${GFLAGS_HOME}/target/lib/Release/gflags_static.lib)

set(SNAPPY_HOME $ENV{THIRDPARTY_HOME}/snappy-visual-cpp)
set(SNAPPY_INCLUDE ${SNAPPY_HOME})
set(SNAPPY_LIB_DEBUG ${SNAPPY_HOME}/x64/Debug/snappy64.lib)
set(SNAPPY_LIB_RELEASE ${SNAPPY_HOME}/x64/Release/snappy64.lib)

set(LZ4_HOME $ENV{THIRDPARTY_HOME}/lz4-1.7.5)
set(LZ4_INCLUDE ${LZ4_HOME}/lib)
set(LZ4_LIB_DEBUG ${LZ4_HOME}/visual/VS2010/bin/x64_Debug/liblz4_static.lib)
set(LZ4_LIB_RELEASE ${LZ4_HOME}/visual/VS2010/bin/x64_Release/liblz4_static.lib)

set(ZLIB_HOME $ENV{THIRDPARTY_HOME}/zlib-1.2.11)
set(ZLIB_INCLUDE ${ZLIB_HOME})
set(ZLIB_LIB_DEBUG ${ZLIB_HOME}/contrib/vstudio/vc14/x64/ZlibStatDebug/zlibwapi.lib)
set(ZLIB_LIB_RELEASE ${ZLIB_HOME}/contrib/vstudio/vc14/x64/ZlibStatRelease/zlibwapi.lib)
And then finally to compile RocksDB:


mkdir build 
cd build 

in build folder create build.cmd containg:

set THIRDPARTY_HOME=%CODE_HOME%
cmake -DJNI=1 -DGFLAGS=1 -DSNAPPY=1 -DLZ4=1 -DZLIB=1 -DXPRESS=1 .. 

5. execute command  build.cmd
6 in visual studio open rocksdb.sln file created by executed command 
5. For all projects change if necessary sdk to latest
6. for rocksdb, tocksdbjni, rocksdbjni-shared in c++/code generation option  disable enchanced instruction set (ahange AVX2 to not set)
7. Change warning level to 2 (W2)
8. build rocksdbjni shared
   any error about missing header, library by compiler or linker etc , indicate wrong configuration in thirdparty.inc , so in that case  delete content of the build folder , fix thirdparty.inc and run build.cmd again
9.AS A RESULT YOU RECEIVE: build\java\rocksdb_jni.jar and build\java\Release\rocksdbjni-shared.dll. jar file does not contain dll , so please copy dll to jar file -- taking as sample archiva version 7.7.2-p2 (need to change names)
10. update version in archiva
   info: please take previous version in archiva as sample. Jar file is imported twice using x64 and win64 clasifier
   info: in archiva import dll  to rocksdbjni/rocksdbjni and jar to org.rocksdb/rocksdbjni 













