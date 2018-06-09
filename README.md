# cmake-sdl2-template
Minimal C++ with SDL2 project using CMake build process management.

## Requirements
CMake is the tool used here in order to manage build processes, so it must be available beforehand.
In Debian-based distributions, CMake is available through the package manager.
However, it is very likely that your OS repositories are not updated with one of CMake's latest version.

So, I would recommend to build it and install it manually if you want the latest stable version.
Otherwise you can just use the package manager for an easier install procedure.

### Ubuntu - Build and install CMake manually (recommended)
Uninstall any previous version (if needed):
```
sudo apt remove cmake
sudo apt purge --auto-remove cmake
```

Download, build and install the desired version from the terminal (you can check for the available versions at [the official CMake webpage](https://cmake.org/download/)). 
You should set the CMAKE_VERSION and CMAKE_BUILD variables as needed:
```
export CMAKE_VERSION=3.11
export CMAKE_BUILD=3
mkdir -p ~/3rdparty
cd ~/3rdparty
wget https://cmake.org/files/v$CMAKE_VERSION/cmake-$CMAKE_VERSION.$CMAKE_BUILD.tar.gz
tar -xvzf cmake-$CMAKE_VERSION.$CMAKE_BUILD.tar.gz
rm cmake-$CMAKE_VERSION.$CMAKE_BUILD.tar.gz
cd cmake-$CMAKE_VERSION.$CMAKE_BUILD
./bootstrap
make -j4
sudo make install
```

### Ubuntu - Install CMake through package manager (easier, but may be a bit outdated)
```
sudo apt-get update
sudo apt-get install cmake
```

### Windows (the easiest of all)
Just run any installer from [the official CMake webpage](https://cmake.org/download/) and you should be good to go.
Personally, and even if this is not a recommended practice, I like to include its bin directory, e.g. C:\Program Files\CMake\bin in the Path environment variable.

## How to use
First of all, clone the repo to a desired location:
```
mkdir -p ~/workspace
cd ~/workspace
git clone git@github.com:azarrias/cmake-sdl2-template.git
```
Then create a directory for the out-of-source build and execute cmake as needed.

In this example, Eclipse will be the IDE of choice:
```
cd ~/workspace/cmake-sdl2-template
mkdir build && cd build
cmake -G "Eclipse CDT4 - Unix Makefiles" -D CMAKE_BUILD_TYPE=Debug -D CMAKE_ECLIPSE_VERSION=4.7.2 ../src 
```
To set up the project in Eclipse:
1. Start Eclipse (it does not matter where you set your workspace).
2. Uncheck the "Project -> Build Automatically" option.
3. Select "Import... -> Existing Projects into Workspace".
4. In the open dialog, select the build path, i.e. ~/workspace/cmake-sdl2-template/build. 
Eclipse should be able to find the project.
5. Click on "Project -> Build Project".

## Windows considerations
If you intend to make a build for Windows, you should know that Windows does not have a predefined location for installing libraries, 
so it's up to you to find out how to help CMake find them.

There is a number of ways to accomplish this, but the least painful way is to follow these steps:
1. Set the SDL2_DIR environment variable. You can set it from the Environment variables dialog in the Control Panel, or straight from the command prompt like so:
```
set SDL2_DIR=c:\libraries
```
2. In the same location that was set for SDL2_DIR, create the sdl2-config.cmake file, with the following content:
```
set(prefix "${CMAKE_CURRENT_LIST_DIR}")
set(exec_prefix "${prefix}")
set(SDL2_PREFIX "${prefix}")
set(SDL2_EXEC_PREFIX "${prefix}")

set(SDL2_INCLUDE_DIRS "${prefix}/include")

if(${CMAKE_SIZEOF_VOID_P} MATCHES 8)
  set(libdir "${prefix}/lib/x64")
  set(SDL2_LIBDIR "${prefix}/lib/x64")
else()
  set(libdir "${prefix}/lib/x86")
  set(SDL2_LIBDIR "${prefix}/lib/x86")  
endif()

set(SDL2_LIBRARIES "${SDL2_LIBDIR}/SDL2.lib;${SDL2_LIBDIR}/SDL2main.lib")
string(STRIP "${SDL2_LIBRARIES}" SDL2_LIBRARIES)
```

You can adjust any of the included paths as needed, but this example of sdl2-config.cmake file will work for the following file tree:
```
libraries/
  +-- include/
        +-- SDL2/
              +-- SDL.h
              +-- SDL_assert.h
              +-- ...
        +-- ...
  +-- lib/
        +-- x64/
              +-- SDL2.dll
              +-- SDL2.lib
              +-- ...
        +-- x86/
              +-- ...
  +-- sdl2-config.cmake
```

## Troubleshooting
There seems to be a problem with the sdl2-config.cmake script provided with SDL2 development libraries.
If you get this error, it can be easily fixed:
```
CMake Error at CMakeLists.txt:18 (add_executable):
  Target "main" links to item "-L/usr/lib/x86_64-linux-gnu -lSDL2 " which has
  leading or trailing whitespace.  This is now an error according to policy
  CMP0004.
```
You just need to find the sdl2-config.cmake file:
```
locate sdl2-config.cmake
```
Once that you know where it is, go and edit this file, removing any extra expaces and replacing this line:
```
set(SDL2_LIBRARIES "-L${SDL2_LIBDIR}  -lSDL2 ")
```
With this:
```
set(SDL2_LIBRARIES "-L${SDL2_LIBDIR}  -lSDL2")
```
