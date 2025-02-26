+++
title = "The Makings of Shute"
date = 2025-02-26T13:17:28+02:00
draft = true
+++

# TL;DR:

Shute (which stems from **sh**ader comp**ute**) is a Rust library I have been developing to simplify the use of compute shaders for GPGPU applications.

Repository: https://github.com/shzhe02/shute (this link should work soon)

# Origins

In the 2nd quarter of 2023, I took a course at my university called [Programming Parallel Computers](https://ppc.cs.aalto.fi/), which remains one of my favorite courses I've ever completed. However, the last chapter, GPU programming, caused my quite a bit of a headache. This was not because of the actual programming exercises, but rather everything else involved in the process of running GPU code. See, at the time, I was mainly doing my exercises on my laptop, which only had integrated graphics. That means using CUDA (the GPU API of choice for the course) on my laptop is basically impossible. Luckily, I had a desktop at home which has a GTX 1060, so I thought I could use it to do my exercises. Unfortunately, I was using Linux (Fedora, specifically) as my operating system, and NVIDIA driver support at the time wasn't great. I *could* have set up CUDA on it, but I preferred to not touch the long commands needed to do so. Yeah, I was pretty lazy.

Luckily, the courses did offer an alternative API: OpenCL. For some background, [this page](https://ppc.cs.aalto.fi/ch4/v0opencl/) is my first impression of OpenCL. OpenCL does offer much wider hardware compatibility, meaning I could run it on all my machines, but the CPU-side code is so much more verbose than the CUDA version (like what even is a command queue, and also why do I need to compile my kernel?). Additionally, [developing applications with OpenCL still requires installing an SDK on Windows](https://github.com/KhronosGroup/OpenCL-Guide/blob/main/chapters/getting_started_windows.md#opencl-sdk).

Surely there must be a better way, right? I mean, games use my GPU just fine, and I don't remember ever having to install drivers or other supplementary software just to get my games running. 

## GPU APIs

As it turns out, there are generally two categories of APIs for GPUs: graphics APIs and compute APIs. OpenCL and CUDA belong to the latter, while APIs such as OpenGL and Vulkan belong to the former. Modern operating systems tend to come with at least one graphics API supported out of the box, which is the best for setup complexity (because there is none). Even better, there exists a Rust library called [wgpu](https://github.com/gfx-rs/wgpu) which abstracts most common graphics APIs into a unified WebGPU-like API.

Okay, now we know that if we can get a GPGPU application built on wgpu, it'll basically run on the vast majority of computers without any setup required. But now, the question is: **how?** 

As it turns out, most modern graphics APIs support something known as **compute shaders**, which serve as the key to creating GPGPU applications on graphics APIs. Aside from a few additional lines of code, compute shaders look quite similar to compute kernels from, e.g., CUDA. The problem? Using compute shaders through graphics APIs leads to CPU-side code that looks even more verbose than OpenCL, seeing as doing even simple stuff with graphics APIs such as drawing a triangle requires around [a thousand lines of code](https://github.com/SaschaWillems/Vulkan/blob/master/examples/triangle/triangle.cpp).

# Development

Safe to say, the verbosity of the code needed to work with graphics APIs spooked me for quite a while... like half a year. I eventually picked up Rust properly during this time, and I also learned the joys of adding additional crates to my project with a nice `cargo add <crate>` (as opposed to requiring tools like CMake).



