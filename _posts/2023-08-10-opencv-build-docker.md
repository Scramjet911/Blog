---
title: "Setting up docker for custom opencv build"
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

### The premise
We made a react native (android focused) package which used the OpenCV Laplacian filter to detect blurring in images. I will write about that in another post.  
So, the client liked the package functionality, but they had a problem. The app size was huge, 150mb on integrating the package into the app.  

We knew that the package would have a large size, since the OpenCV libs were that large. Just the opencv aar package was 75mb.  
Now, the only way to reduce it's size was to take a custom build of OpenCV, since it made up the bulk of the app size. (I'm getting excited just thinking about this....)  
I love getting challenging tasks 😎   

***This is part 1 of the series for a custom opencv build.  
Part 1 is mainly about the docker env, one can do it locally (by installing the same dependencies and changing RUN for sudo), but I didn't want to take a chance on dependency conflicts (If I needed multiple versions and all).***

### The target
So I had a week, to setup the C++ environment and build OpenCV. Now, I have heard that there are dependency conflicts (usually) when building C++ programs (because of system dependencies clashing with required dependencies). Now take everything I say with a pinch of salt because this is the first time I am cross compiling a software.  

So the first thing to do is to avoid dependency conflicts, the first thing that came to mind was a docker container.  
It is independent of the file system, and acts like a virtual environment (obviously it is much more capable than that) for me.  
Thus I built a container for it.  
I decided to go for an ubuntu base, since after a lot of research I found that alpine doesn't have support (from what I could find) for multilib, since it uses `musl` and not `glibc`(used by ubuntu). Therefore it is easier to use the ubuntu base, although the size of the image would increase.  

Next we need to install the dependencies. So the dockerfile would look like this

```dockerfile
FROM ubuntu

# Set the working directory
WORKDIR /app

COPY ./ndk/android-ndk-r26c /app/ndk

# ---------------------------- Build dependencies install -----------------------------------
# Installing dependencies, might have some unnecessary ones but didn't have time to check each one
RUN apt-get -y update
RUN --mount=type=cache,target=/var/cache/apt \
    apt-get install -y build-essential checkinstall cmake pkg-config yasm \
    git gfortran \
    libjpeg8-dev libpng-dev libtiff5-dev \
    libavcodec-dev libavformat-dev libswscale-dev libdc1394-dev \
    libxine2-dev libv4l-dev

RUN --mount=type=cache,target=/var/cache/apt \
    apt-get install -y gstreamer1.0-gl gstreamer1.0-opencv \
    gstreamer1.0-plugins-bad gstreamer1.0-plugins-good \
    gstreamer1.0-plugins-ugly gstreamer1.0-tools \
    libgstreamer-plugins-base1.0-dev \
    libgstreamer1.0-0 libgstreamer1.0-dev

RUN --mount=type=cache,target=/var/cache/apt \
    apt-get install -y libgtk2.0-dev libgtk-3-dev libtbb-dev \
    libatlas-base-dev \
    libfaac-dev libmp3lame-dev libtheora-dev \
    libvorbis-dev libxvidcore-dev \
    libopencore-amrnb-dev libopencore-amrwb-dev \
    x264 v4l-utils

# Can remove python-is-python3 if python becomes default in ubuntu
RUN --mount=type=cache,target=/var/cache/apt \
    apt-get install -y qt6-base-dev \
    apt-get install -y ninja-build ant python-is-python3

```
One might wonder what the `--mount=type=cache,target=/var/cache/apt` part of the run command does, since `Run` commands are cached by default. Well 

This takes care of all the dependencies needed to build just opencv (It is a lot, I know).  
We also need to install android build tools, needed to compile opencv for android. 
Thank you [@thyrlian](https://github.com/thyrlian) for [this script](https://github.com/thyrlian/AndroidSDK/blob/master/android-sdk/license_accepter.sh)  
I shamelessly copied it from his `android-sdk` docker image.

```dockerfile

# ---------------------------- Android sdk install -----------------------------------
# support multiarch: i386 architecture
# install Java
# install essential tools
# install Qt
ARG JDK_VERSION=17
RUN dpkg --add-architecture i386 && \
    apt-get -y update

RUN --mount=type=cache,target=/var/cache/apt \
    apt-get dist-upgrade -y && \
    apt-get install -y --no-install-recommends libncurses5:i386 libc6:i386 libstdc++6:i386 lib32ncurses6 lib32z1 zlib1g:i386 && \
    apt-get install -y --no-install-recommends openjdk-${JDK_VERSION}-jdk && \
    apt-get install -y --no-install-recommends git wget unzip && \
    DEBIAN_FRONTEND=noninteractive

# download and install Gradle
# https://services.gradle.org/distributions/
ARG GRADLE_VERSION=8.2
ARG GRADLE_DIST=bin
RUN cd /opt && \
    wget -q https://services.gradle.org/distributions/gradle-${GRADLE_VERSION}-${GRADLE_DIST}.zip && \
    unzip gradle*.zip && \
    ls -d */ | sed 's/\/*$//g' | xargs -I{} mv {} gradle && \
    rm gradle*.zip

# download and install Kotlin compiler
# https://github.com/JetBrains/kotlin/releases/latest
ARG KOTLIN_VERSION=1.9.20
RUN cd /opt && \
    wget -q https://github.com/JetBrains/kotlin/releases/download/v${KOTLIN_VERSION}/kotlin-compiler-${KOTLIN_VERSION}.zip && \
    unzip *kotlin*.zip && \
    rm *kotlin*.zip

# download and install Android SDK
# https://developer.android.com/studio#command-tools
ARG ANDROID_SDK_VERSION=11076708
ENV ANDROID_SDK_ROOT /opt/android-sdk
RUN mkdir -p ${ANDROID_SDK_ROOT}/cmdline-tools && \
    wget -q https://dl.google.com/android/repository/commandlinetools-linux-${ANDROID_SDK_VERSION}_latest.zip && \
    unzip *tools*linux*.zip -d ${ANDROID_SDK_ROOT}/cmdline-tools && \
    mv ${ANDROID_SDK_ROOT}/cmdline-tools/cmdline-tools ${ANDROID_SDK_ROOT}/cmdline-tools/tools && \
    rm *tools*linux*.zip

# set the environment variables
ENV JAVA_HOME /usr/lib/jvm/java-${JDK_VERSION}-openjdk-amd64
ENV GRADLE_HOME /opt/gradle
ENV KOTLIN_HOME /opt/kotlinc
ENV PATH ${PATH}:${GRADLE_HOME}/bin:${KOTLIN_HOME}/bin:${ANDROID_SDK_ROOT}/cmdline-tools/latest/bin:${ANDROID_SDK_ROOT}/cmdline-tools/tools/bin:${ANDROID_SDK_ROOT}/platform-tools:${ANDROID_SDK_ROOT}/emulator
# WORKAROUND: for issue https://issuetracker.google.com/issues/37137213
ENV LD_LIBRARY_PATH ${ANDROID_SDK_ROOT}/emulator/lib64:${ANDROID_SDK_ROOT}/emulator/lib64/qt/lib
# patch emulator issue: Running as root without --no-sandbox is not supported. See https://crbug.com/638180.
# https://doc.qt.io/qt-5/qtwebengine-platform-notes.html#sandboxing-support
ENV QTWEBENGINE_DISABLE_SANDBOX 1

# accept the license agreements of the SDK components
ADD license_accepter.sh /opt/
RUN chmod +x /opt/license_accepter.sh && /opt/license_accepter.sh $ANDROID_SDK_ROOT

# setup adb server
EXPOSE 5037

# Installing sdk build tools for target sdk version. We are using Android 33
RUN sdkmanager --install "build-tools:33.0.3"

```

And finally.. What we all were waiting for...  
Oh-my-zsh support 😜. We can't be using the crummy old regular terminal amiright?  
Well actually I installed it because I needed the autocomplete for the commands I ran.  
One can skip it if not needed.

```dockerfile
RUN sh -c "$(wget -O- https://github.com/deluan/zsh-in-docker/releases/download/v1.1.5/zsh-in-docker.sh)" -- \
    -t amuse \
    -p git \
    -p https://github.com/zsh-users/zsh-autosuggestions \
    -p https://github.com/zsh-users/zsh-syntax-highlighting 

# setup env variables
ENV NDK /app/ndk
ENV ANDROID_HOME /opt/android-sdk/cmdline-tools


COPY ./opencv /app/opencv
COPY build_custom_cv.sh /app

# Run bash shell when the container starts
CMD ["/bin/zsh"]

```

We're done with setting up the container!
[Read on]({{ site.baseurl }}{% post_url 2023-08-11-opencv-build-cmake %}) to for the custom build steps