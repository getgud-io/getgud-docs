# Build Getgud C++ SDK

Getgud C++ SDK allows you to integrate your game with the Getgud platform. Once integrated, you will be able to stream your matches to Getgud's cloud, as well as to send reports and update player's data. This README will include all possible build instructions for different systems.


<b>[IMPORTANT]</b> Before you start clone Getgud SDK repo **recursively**
```
git clone https://github.com/getgud-io/cpp-getgud-sdk-dev --recursive
cd cpp-getgud-sdk-dev
```

<b>[IMPORTANT]</b> When Getgud SDK is running it depends on libcurl and zlib, make sure they are installed on the system.
## Table of Contents

- [Build for Linux](https://github.com/getgud-io/getgud-docs/blob/main/1-Integrations/cpp-build-instructions.md#build-for-linux)
- [Build for Windows](https://github.com/getgud-io/getgud-docs/blob/main/1-Integrations/cpp-build-instructions.md#build-for-windows)
- [Build for Unreal Engine](https://github.com/getgud-io/getgud-docs/blob/main/1-Integrations/cpp-build-instructions.md#build-for-unreal-engine)
- [Build for Unity](https://github.com/getgud-io/getgud-docs/blob/main/1-Integrations/cpp-build-instructions.md#build-for-unity)

## Build for Linux

### Requirements:
- CMake version >= 3.22.1
- make
- gcc and g++
- libcurl development tools
- libssl
- libcrypto
- openssl development tools

Before building install all required libraries
```
sudo apt update
sudo apt-get install git binutils make csh g++ sed gawk autoconf automake autotools-dev shtool libtool curl cmake libssl-dev libpsl-dev
```

Please make sure that the version of your CMake is >= 3.22.1 by using
```
cmake --version
```
If your CMake version is older than >= 3.22.1, update it to a newer version.

To start the build process run our predefined script
```
cd ./cpp-getgud-sdk-dev/tools
sh linuxbuild_so.sh
```

Congrats, SDK is built! You will mostly need `getgudsdk.a` and `getguddsk.so` files for Linux.
 
## Build for Windows

### Requirements:
- [CMake](https://cmake.org/download/)
- [make, available through Visual Studio](https://visualstudio.microsoft.com/downloads/)
- x64 Native Tools Command Prompt, available through Visual Studio

<b>[IMPORTANT]</b> All commands should run in **x64 Native Tools Command Prompt**

To start the build process run our predefined script
```
cd ./cpp-getgud-sdk-dev/tools
winbuild_dll.bat
```

Congrats, SDK is built! Your build files should be located in `build/_buid/Release` folder.

## Build for Unreal Engine

To build and integrate for UE follow [this tutorial](https://github.com/getgud-io/getgud-docs/blob/main/1-Integrations/Unreal%20Engine/unreal-engine-integration.md)


## Build for Unity

<b>Windows</b>: If you are planning to use Getgud SDK on Windows follow our [Windows Build](https://github.com/getgud-io/getgud-docs/blob/main/1-Integrations/cpp-build-instructions.md#build-for-windows) instructions.

<b>Linux</b>: If you are planning to use Getgud SDK on Linux:

Before building install all required libraries
```
sudo apt update
sudo apt-get install git binutils make csh g++ sed gawk autoconf automake autotools-dev shtool libtool curl cmake libssl-dev libpsl-dev
```

Please make sure that the version of your cmake is >= 3.22.1 by using
```
cmake --version
```
If your CMake version is older than >= 3.22.1, update it to a newer version.

To start the build process run our predefined script
```
cd ./cpp-getgud-sdk-dev/tools
sh linux_so_Unity_UE.sh
```

Once you have built SDK follow this [Unity integration](https://github.com/getgud-io/getgud-docs/blob/main/1-Integrations/Unity/unity-integration.md) tutorial.

