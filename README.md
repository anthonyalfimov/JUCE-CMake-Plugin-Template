# JUCE CMake Audio Plugin Template

[![Build](https://img.shields.io/github/actions/workflow/status/anthonyalfimov/JUCE-CMake-Plugin-Template/Validation.yml?branch=main&logo=github)](https://github.com/anthonyalfimov/JUCE-CMake-Plugin-Template/actions)

A template for creating an audio plugin using [JUCE 6/7](https://github.com/juce-framework/JUCE) and [CMake](https://cmake.org).

- Works as a drop-in replacement for Projucer - no changes to the source code are necessary! The template can also be used alongside a `.jucer` project.
- Generates clean Xcode and Visual Studio projects (reasonable source file organisation, only the necessary build schemes for Xcode).
- Uses CMake to manage dependencies (e.g. JUCE). The template creates a shallow clone of the specified git tag or branch to reduce download times and disk usage.
- Uses GitHub Actions to build and validate the plugin on MacOS and Windows. Dependencies and compiler output are cached for faster builds.

To learn how to replace a Projucer project with this template, see the [**Migrating from Projucer**](MIGRATE_FROM_PROJUCER.md) guide.

## Generating IDE project

To generate an **Xcode** project, run:
```sh
cmake -B Build -G Xcode -D CMAKE_OSX_ARCHITECTURES=arm64\;x86_64 -D CMAKE_OSX_DEPLOYMENT_TARGET=10.13
```
The `-D CMAKE_OSX_ARCHITECTURES=arm64\;x86_64` flag is required to build universal binaries.

The `-D CMAKE_OSX_DEPLOYMENT_TARGET=10.13` flag sets the minimum MacOS version to be supported.

---

To generate a **Visual Studio 2022 (17)** project, run:
```sh
cmake -B Build -G "Visual Studio 17"
```

## Building

To build the generated IDE project from the command line, run:
```sh
cmake --build Build --config Debug
```

## References

Based on the [JUCE/examples/CMake/AudioPlugin](https://github.com/juce-framework/JUCE/tree/master/examples/CMake/AudioPlugin) template.

Inspired by:

- [sudara/pamplejuce](https://github.com/sudara/pamplejuce)
- [eyalamirmusic/JUCECmakeRepoPrototype](https://github.com/eyalamirmusic/JUCECmakeRepoPrototype)
