---
title: "Rusting away my time"
categories: 
    - ML
    - WASM
    - Rust
tags: 
    - Rust
    - Federated learning
    - Burn
    - Android
header: 
sidebar:
    nav : hobby
---

In my journey through searching for federated learning methods other than tflite (don't blacklist me Google), I came upon [Burn by traceAI](https://github.com/tracel-ai/burn) which is a machine learning framework in Rust.

Burn has inference and training capabilities, which makes it ideal to implement on device learning. 
The only question that remains is if I can compile it for wasm or for that matter, android...

Getting sidetracked, I have used C++ in android and even though I faced minor hiccups (like memory leaks), It was pretty easy to build the app since the compatibility is baked into android studio. Rust doesn't have this support, so I have to manually add gradle scripts for the compilation and copying of the built Rust libraries or do it manually, build individually and copy the results. I chose the second option, since I haven't used gradle much and didn't want to get too deep into it.

The way to do this is to add rust compiler flags with arguments to the ndk library in the cargo config like this:
```toml
[target.aarch64-linux-android]
linker = "<path_to_ndk>/android-ndk-r27/toolchains/llvm/prebuilt/linux-x86_64/bin/aarch64-linux-android<android_target>-clang"

[target.armv7-linux-androideabi]
linker = "<path_to_ndk>/android-ndk-r27/toolchains/llvm/prebuilt/linux-x86_64/bin/armv7a-linux-androideabi<android_target>-clang"

[target.i686-linux-android]
linker = "<path_to_ndk>/android-ndk-r27/toolchains/llvm/prebuilt/linux-x86_64/bin/i686-linux-android<android_target>-clang"
```
Add these compiler targets to rutup
```bash
rustup target add aarch64-linux-android armv7-linux-androideabi i686-linux-android
```

Then we just need to build the library for these targets

```bash
cargo build --target aarch64-linux-android --release   
cargo build --target armv7-linux-androideabi --release
cargo build --target i686-linux-android --release
```

Tada!! The built lib files can be found in the build folder.

I made a script (as usual) to do these steps for me (to always automate repetitive tasks is my mantra)..

```bash 
#!/bin/zsh

# Script, if you are building and moving the jni lib's manually
cargo build --target aarch64-linux-android --release
cargo build --target armv7-linux-androideabi --release
cargo build --target i686-linux-android --release
cargo build --target x86_64-linux-android --release

rm -rf ../jniLibs
mkdir -p ../jniLibs/arm64-v8a
mkdir ../jniLibs/armeabi-v7a
mkdir ../jniLibs/x86
mkdir ../jniLibs/x86_64

cp ./target/aarch64-linux-android/release/libmnist_inference_android.so ../jniLibs/arm64-v8a/libmnist_inference_android.so
cp ./target/armv7-linux-androideabi/release/libmnist_inference_android.so ../jniLibs/armeabi-v7a/libmnist_inference_android.so
cp ./target/i686-linux-android/release/libmnist_inference_android.so ../jniLibs/x86/libmnist_inference_android.so
cp ./target/x86_64-linux-android/release/libmnist_inference_android.so ../jniLibs/x86_64/libmnist_inference_android.so
```

So I just have to run this script when I want to update my `.so` files with the latest code.  
(Later I actually used a gradle plugin to do this - [rust-android-gradle](https://github.com/mozilla/rust-android-gradle) which made things simpler, [check it out](https://github.com/Scramjet911/burn/tree/mnist-android-example/examples/mnist-inference-android))


Okay, let's do the jni bridging in [Part 2]({{ site.baseurl }}{% post_url 2024-07-22-rusting-away-2 %})