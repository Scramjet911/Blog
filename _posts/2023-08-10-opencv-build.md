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

### The premise
We made a react native (android focused) package which used OpenCV Laplacian filter to detect blurring in images. Will write about that sometime later I guess.  
The client liked the package functionality but they had a problem. The app size was huge, 150mb on integrating the package into the app.  

We knew that the package would have a large size, since the OpenCV libs were that large. Just the opencv aar package was 75mb.  
The only way to reduce the size was to take a custom build of OpenCV. (I'm getting excited just thinking about this....)  
I love getting challenging tasks 😎   

### The target
So I had a week, to setup the C++ environment and build OpenCV. Now, I have heard that there are dependency conflicts (usually) when building C++ programs.  
To avoid that I decided to first test out building opencv in a docker container.