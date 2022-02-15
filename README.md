# JUCE CMake Audio Plugin Template

![Validation](https://img.shields.io/github/workflow/status/anthonyalfimov/JUCE-CMake-Plugin-Template/Validation/main?label=Validation&logo=github)

A template for creating an audio plugin using [JUCE 6](https://github.com/juce-framework/JUCE) and [CMake](https://cmake.org).

- Generates clean Xcode projects (only the necessary build schemes, reasonable source file organisation).
- Uses CMake to manage dependencies (e.g. JUCE). The template creates a shallow clone of the specified git tag or branch to reduce download times and disk usage.
- Uses GitHub Actions to build and validate the plugin on MacOS and Windows. Dependencies and compiler output are cached for faster builds.

## Generating IDE project

To generate an Xcode project, run:
```sh
cmake -B Build -G Xcode -D CMAKE_OSX_ARCHITECTURES=arm64\;x86_64
```
The `-D CMAKE_OSX_ARCHITECTURES=arm64\;x86_64` flag is required to build universal binaries.

## Building

To build the Xcode project from the command line, run:
```sh
cmake --build Build --config Debug -- -quiet
```
Everything after `--` is passed to the native build tool. The `-quiet` flag suppresses excessively verbose `xcodebuild` output.

## References

Based on the [JUCE/examples/CMake/AudioPlugin](https://github.com/juce-framework/JUCE/tree/master/examples/CMake/AudioPlugin) template.

Inspired by:

- [sudara/pamplejuce](https://github.com/sudara/pamplejuce)
- [eyalamirmusic/JUCECmakeRepoPrototype](https://github.com/eyalamirmusic/JUCECmakeRepoPrototype)