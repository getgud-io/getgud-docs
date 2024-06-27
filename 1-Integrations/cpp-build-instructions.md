# Build Getgud C++ SDK

Getgud C++ SDK allows you to integrate your game with the GetGud platform. Once integrated, you will be able to stream your matches to Getgud's cloud, as well as to send reports and update player's data. This is a development repository which contains all our private code for SDK development. This README will include all possible build instructions for different systems. To check all SDK functions and how it works you can visit one of the client repos with the docs.


<b>[IMPORTANT]</b> Before you start clone Getgud SDK repo **recursively**
```
git clone https://github.com/getgud-io/cpp-getgud-sdk-dev --recursive
cd cpp-getgud-sdk-dev
```

## Table of Contents

- [Build for Linux](https://github.com/getgud-io/getgud-docs/blob/main/1-Integrations/cpp-build-instructions.md#build-for-linux)
- [Build for Windows](https://github.com/getgud-io/getgud-docs/blob/main/1-Integrations/cpp-build-instructions.md#build-for-windows)
- [Build for Unreal Engine](https://github.com/getgud-io/getgud-docs/blob/main/1-Integrations/cpp-build-instructions.md#build-for-unreal-engine)
- [Build for Unity](https://github.com/getgud-io/getgud-docs/blob/main/1-Integrations/cpp-build-instructions.md#build-for-unity)

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

To start the build process run our predefined script
```
cd cpp-getgud-sdk-dev
sh tools/linuxbuild_so.sh
```

Congrats, SDK is built! You will mostly need `getgudsdk.a` and `getguddsk.so` files for Linux.
 
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

## Build for Unreal Engine

To build and integrate for UE follow [this tutorial](https://github.com/getgud-io/getgud-docs/blob/main/1-Integrations/Unreal%20Engine/unreal-engine-integration.md)


## Build for Unity

1. To build and integrate for Unity you should build SDK for your Windows/Linux server as described in our tutorials.
2. Integrate SDK build files into Unity as described in [this tutorial](https://github.com/getgud-io/getgud-docs/blob/main/1-Integrations/Unity/unity-integration.md)


