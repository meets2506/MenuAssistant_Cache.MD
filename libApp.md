# libApp Project

## Overview
libApp is a cross-platform library designed for efficient data processing and communication, particularly targeting Android and ARM architectures. It includes components for HTTP communication, BM25 indexing, and server functionality.

## File Structure
- **Android.mk**: Makefile for Android builds.
- **Application.mk**: Android application configuration.
- **CMakeLists.txt**: CMake configuration for cross-platform builds.
- **Makefile**: Main Makefile for building the project.
- **Makefile.android**: Makefile specifically for Android builds.
- **Makefile.arm**: Makefile for ARM architecture builds.
- **SOURCE/**: Contains the core source code files.
- **android_http_client.cpp/h**: HTTP client implementation for Android.
- **bm25.cpp/h**: BM25 algorithm implementation for indexing.
- **lib_app_server.cpp/h**: Server implementation for the library.
- **build_android.sh**: Script to build the project for Android.
- **build_android_arm.sh**: Script to build the project for Android on ARM.

## Cross-Compilation
Cross-compilation is handled using the Android NDK and CMake. The `build_android.sh` and `build_android_arm.sh` scripts automate the process for Android and ARM targets.

## Build Instructions
1. **Android Build**: Run `./build_android.sh`.
2. **Android ARM Build**: Run `./build_android_arm.sh`.
3. **General Build**: Use `make` with the appropriate Makefile.

## Usage
Include the library in your project and use the provided APIs for HTTP communication, BM25 indexing, and server functionality.
