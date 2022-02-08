# JUCE CMake Audio Plugin Template

A template for creating an audio plugin using JUCE6 and CMake.

- Generates clean Xcode projects (only the necessary build schemes, reasonable source file organisation).
- Uses CMake to manage dependencies (e.g. JUCE). The template creates a shallow clone of the specified git tag or branch to reduce download times and disk usage.

## Generating IDE project

To generate an Xcode project, run:
```sh
cmake -B Build -G Xcode
```

## References

Based on the [JUCE/examples/CMake/AudioPlugin](https://github.com/juce-framework/JUCE/tree/master/examples/CMake/AudioPlugin) template.

Inspired by:
- [sudara/pamplejuce](https://github.com/sudara/pamplejuce)
- [eyalamirmusic/JUCECmakeRepoPrototype](https://github.com/eyalamirmusic/JUCECmakeRepoPrototype)