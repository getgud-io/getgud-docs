# Build Getgud C++ SDK

Getgud C++ SDK allows you to integrate your game with the GetGud platform. Once integrated, you will be able to stream your matches to Getgud's cloud, as well as to send reports and update player's data. This is a development repository which contains all our private code for SDK development. This README will include all possible build instructions for different systems. To check all SDK functions and how it works you can visit one of the client repos with the docs.

## Table of Contents

- [Build for Linux](https://github.com/getgud-io/getgud-docs/blob/main/Integrations/cpp-build-instructions.md#build-for-linux)
- [Build for Windows](https://github.com/getgud-io/getgud-docs/blob/main/Integrations/cpp-build-instructions.md#build-for-windows)

## Build for Linux

### Requirements:
- CMake 3.18
- make
- gcc and g++
- libcurl development tools
- libssl
- libcrypto
- zlib
- openssl development tools

First we need to build libcurl and zlib which are used inside SDK

### libcurl
```bash *Debian*
sudo apt-get install binutils make csh g++ sed gawk autoconf automake autotools-dev shtool libtool curl cmake
cd libs/libcurl/
./buildconf
./configure --disable-shared --with-openssl --prefix=FULL_PATH_TO_SDK/libs/libcurl/builds/libcurl-x64-debug-static --enable-debug
./configure --disable-shared --with-openssl --prefix=FULL_PATH_TO_SDK/libs/libcurl/builds/libcurl-x64-release-static
```
If you do not need release build you can remove the last ./configure command
Replace `FULL_PATH_TO_SDK` to your full system path to `cpp-getgud-sdk-dev` folder!

Next do from the each building folder
```bash
make
make install
```
The build files will appear in the build folder, you will need mostly `libcurl.a` and `libcurl.so` for Linux!


### zlib
```bash
cd libs/zlib/
./configure -prefix=./
make
make install 
```

The build files will appear in the root folder of zlib folder, you will need mostly `zlib.a` and `zlib.so` for Linux! 


### Build SDK

Now that we have build libraries that we need, let's build SDK itself.
This build sdk .a file
```bash
cmake --no-warn-unused-cli -DCMAKE_EXPORT_COMPILE_COMMANDS:BOOL=TRUE -DCMAKE_BUILD_TYPE:STRING=Release -DCMAKE_C_COMPILER:FILEPATH=/usr/bin/gcc -DCMAKE_CXX_COMPILER:FILEPATH=/usr/bin/g++ -SFULL_PATH_TO_SDK -BFULL_PATH_TO_SDK/build -G "Unix Makefiles"
cd build 
cmake --build FULL_PATH_TO_SDK/build --config Release --target all -j 4 --
```

And this builds sdk .so file
Rm all from build folder except _build

```bash
cmake --no-warn-unused-cli -DSO_BUILD:BOOL=TRUE -DCMAKE_EXPORT_COMPILE_COMMANDS:BOOL=TRUE -DCMAKE_BUILD_TYPE:STRING=Release -DCMAKE_C_COMPILER:FILEPATH=/usr/bin/gcc -DCMAKE_CXX_COMPILER:FILEPATH=/usr/bin/g++ -SFULL_PATH_TO_SDK -BFULL_PATH_TO_SDK/build -G "Unix Makefiles"
cmake --build FULL_PATH_TO_SDK/build --config Release --target all -j 4 --
```

Replace `FULL_PATH_TO_SDK` with your full system path to SDK. Example: `/home/admin/cpp-getgud-sdk-dev`

Congrats, SDK is built! You will mostly need `getgudsdk.a` and `getguddsk.so` files for Linux. Sometime you will also need to use build files for zlib and libcurl that we created
 
## Build for Windows

### Requirements:
- CMake 3.18
- libcurl
- zlib

First we need to build libcurl and zlib which are used inside SDK

### libcurl
From the root directroty of libcurl
```bash
set RTLIBCFG=static
call .\buildconf.bat
cd .\winbuild
call nmake /f Makefile.vc mode=static vc=17 debug=yes
call nmake /f Makefile.vc mode=static vc=17 debug=no
```

### zlib
From the root directroty of zlib
```bash
	call nmake /f win32/Makefile.msc
```

Now that we have build libraries that we need, let's build SDK itself.
This build sdk .lib file
```bash
cmake -DDLL_BUILD:STRING=False ..
cmake --build .
cmake --build . --config Release
```

And this builds sdk .dll file
Rm all from build folder except _build

```bash
cmake -DDLL_BUILD:STRING=True ..
cmake --build .
cmake --build . --config Release
```

Congrats, SDK is built! You will mostly need `getgudsdk.a` and `getguddsk.so` files for Linux. Sometime you will also need to use build files for zlib and libcurl that we created

## C# build for Linux (the test project only)

The SDK is ready to use on C# with the shared library build only, no more required.

### Requirements:
- dotnet >= 6.0.
- dotnet runtime.
- the requirements for C language build on Linux.

### Building process:
Would be better to use Visual Studio Code with extensions:
- C# Dev Kit.
- C/C++ extension pack.
- CMake.

1. Build the C GetGudSdk shared library.
2. Make sure that the path of the dllPath variable of the GetGudSdk_calls.cs is correct.
3. Make generated the project's files (obj, bin)
4. Build the test project.

### Recomendations:
The test project shouldn't show any runtime error.

dotnet could be installed using offical instructions:
https://learn.microsoft.com/en-us/dotnet/core/install/linux

Mono and mono-complete isn't recommended, but could be used to.

The test project can be build using csc or msc commands.
