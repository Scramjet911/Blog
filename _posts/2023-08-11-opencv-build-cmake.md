---
title: "Custom builds are always a chore"
categories: 
    - Android
    - Opencv
tags: 
    - Android
    - Opencv
    - kotlin
    - C++
    - humble beginnings
header: 
sidebar:
    nav : job
---

Okay, so now we have a ready dockerfile for the android opencv build. Next we just need to run `CMake` and compile the library with `make`.  
But scramjet, you say, "why do we need to call two programs to compile a cpp library? Don't we usually just run the C/C++ file through a `gcc/g++` compiler and use the output?"  
Well the answer is twofold.  
Quoting from [this stackoverflow](https://stackoverflow.com/a/25790020) answer 
> Make (or rather a Makefile) is a buildsystem - it drives the compiler and other build tools to build your code.
>
> CMake is a generator of buildsystems. It can produce Makefiles, it can produce Ninja build files, it can produce KDEvelop or Xcode projects, it can produce Visual Studio solutions. From the same starting point, the same CMakeLists.txt file. So if you have a platform-independent project, CMake is a way to make it buildsystem-independent as well.  

And one needs to know the difference between a compiler(gcc) and a build tool(make). That can be found [here](https://stackoverflow.com/a/768377)


> gcc compiles and/or links a single file. It supports multiple languages, but does not knows how to combine several source files into a non-trivial, running program - you will usually need at least two invocations of gcc (compile and link) to create even the simplest of programs.
>
>Wikipedia page on GCC describes it as a "compiler system":
>
> > The GNU Compiler Collection (usually shortened to GCC) is a compiler system produced by the GNU Project supporting various programming languages.
>
> make is a "build tool" that invokes the compiler (which could be gcc) in a particular sequence to compile multiple sources and link them together. It also tracks dependencies between various source files and object files that result from compilation of sources and does only the operations on components that have changed since last build.
> > Make is a tool which controls the generation of executables and other non-source files of a program from the program's source files.
> > Make gets its knowledge of how to build your program from a file called the makefile, which lists each of the non-source files and how to compute it from other files.

Now that we know what we are going to do, we just need to figure out how to do it....  
Well that's easy, you can just google 'How to build opencv for android' right? Eh, not so much.
At the time of doing this, there was only [one article](https://web.archive.org/web/20230529193547/https://learnopencv.com/install-opencv-on-android-tiny-and-optimized/) (wayback machine version) I could find, from 2018 that showed up in the search with the steps. But most of it was outdated and I needed to build with newer versions of android toolchain...

Well the good thing was that the steps had only reduced. After a lot of google each and every step along the way I figured out that we just need to call `cmake` from the opencv repository with the build path given as an argument.

Thus we make a folder for the build files (in the cloned opencv directory) and move into it,
```
mkdir build && cd build
```
Then call `cmake`

```bash
cmake \
    -DCMAKE_TOOLCHAIN_FILE=$NDK/build/cmake/android.toolchain.cmake \
    -DANDROID_ABI=$ABI \
    -DANDROID_PLATFORM=android-33 \
    -DBUILD_TESTS=OFF \
    -DBUILD_PERF_TESTS=OFF \
    -DBUILD_EXAMPLES=OFF \
    -DBUILD_ANDROID_EXAMPLES=OFF \
    -DBUILD_opencv_apps=OFF \
    -DBUILD_opencv_calib3d=OFF \
    -DBUILD_opencv_features2d=OFF \
    -DBUILD_opencv_dnn=OFF \
    -DBUILD_opencv_video=OFF \
    -DBUILD_opencv_videoio=OFF \
    -DBUILD_opencv_flann=OFF \
    -DBUILD_opencv_objdetect=OFF \
    -DBUILD_opencv_ml=OFF \
    -DANDROID_STL=c++_shared ..
```
- I am not building the video and other ml libraries since I don't need them. 
- The `..` gives the path to the opencv directory. 
- Give your `ABI` version for whichever version you want to build. Each would need to be called separately in different folders since the build files would be different. Possible ones : `'armeabi-v7a', 'arm64-v8a', 'x86_64', 'x86'` etc.  
- Download the ndk from android studio homepage and give the path to the variable `NDK`.

Finally you need to use your respective build tool, I used `make`.

```bash
make -j $nproc
```

Here it builds opencv with the available cores (given by `nproc`).

The library `.so` files should have (cross your fingers) been generated in the `jni/<abi_version>/libopencv_java4.so`

Done!!! You have a custom build of opencv according to your needs. Copy it out from the dockerfile and use it as you need. (Might have a blog for that too in the future)