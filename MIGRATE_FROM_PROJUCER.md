# Migrating from Projucer

This guide will walk you through the steps necessary to substitute an existing Projucer project with the [JUCE CMake Audio Plugin Template](https://github.com/anthonyalfimov/JUCE-CMake-Plugin-Template).

- If you want to **replace** a Projucer project with the CMake template, no changes to your source code are needed!

- If you want to set up the CMake template **alongside** a Projucer project, only minor modifications to your source code will be required.


## Contents
1. [Add template files to your project](#1-add-template-files-to-your-project)
2. [Update `.gitignore`](#2-update-gitignore)
3. [Edit `CMakeLists.txt`](#3-edit-cmakeliststxt)
4. [Organise source files into directories](#4-organise-source-files-into-directories)
5. [Add `CMakeLists.txt` to source file subdirectories](#5-add-cmakeliststxt-to-source-file-subdirectories)
6. [Use CMake to generate the build system](#6-use-cmake-to-generate-the-build-system)
7. ["Unused parameter" warnings](#7-unused-parameter-warnings)

---

### 1. Add template files to your project

Copy these directory and file from the root of the template to the root of your project:

```
.github/
CMakeLists.txt
```

- `.github/` directory contains the GitHub Actions workflow `.github/workflows/Validation.yml`
that builds and validates your plugin.

- `CMakeLists.txt` file contains the project configuration for CMake.

---

> If you are using Finder on MacOS, you will need to reveal hidden files to copy the `.github` directory.
Press `Cmd + Shift + .` to show hidden files, then press `Cmd + Shift + .` to hide them again once you're done.

---

### 2. Update `.gitignore`

Add the following lines to your `.gitignore` file:
```gitignore
**/[Bb]uild*/
/Libs/
```

Alternatively, you can just use the `.gitignore` file supplied with the template.

---

### 3. Edit `CMakeLists.txt`

Update the supplied `CMakeLists.txt` file with your project's information.

Edit the following line to set the name and the version of your project:
```cmake
project(PluginTemplate VERSION 0.0.1)
```
Note that the name of the project is also used as the target name in this template.

---

The template configures CMake to download a project-specific version of JUCE in the `Libs/` directory.
Shallow cloning is used to reduce the download size.

Edit the following line to set the version of JUCE you want to use for your project:
```cmake
set(LIB_JUCE_TAG "6.1.5")
```
You can use either a git tag name, or a git branch name.

- If you use a tag ([list of JUCE tags](https://github.com/juce-framework/JUCE/tags)), your project will be "pinned" to the corresponding version of JUCE. JUCE will only be updated if you change the tag.
- If you use a branch name, JUCE will be updated every time you generate or update the build system with CMake.

---

Edit the arguments of the `juce_add_plugin()` function:
```cmake
juce_add_plugin("${PROJECT_NAME}"
    ...
    <PROPERTY>: <value>
    ...)
```
This is where you set the properties of your plugin (e.g. formats, plugin and manufacturer codes). The template includes the commonly used properties, and you can consult the [JUCE CMake Documentation](https://github.com/juce-framework/JUCE/blob/master/docs/CMake%20API.md#juce_add_target) for the full list of available options.

---

Edit the following line to change the C++ language standard used by your project:
```cmake
target_compile_features("${PROJECT_NAME}" PUBLIC cxx_std_17)
```

---

Uncomment the following line to enable `JuceHeader.h` generation:
```cmake
juce_generate_juce_header("${PROJECT_NAME}")
```
`JuceHeader.h` contains `#include` statements for all JUCE modules used by your project. Generating `JuceHeader.h` is required for Projucer compatibility. Otherwise, JUCE module headers can be included directly in source files that require them.

---

> `JuceHeader.h` is generated when the project is built. Therefore, you need to build the project at least once before your IDE can properly identify JUCE symbols.
> Including JUCE module headers directly avoids this inconvenience.

---

By default, the generated `JuceHeader.h` includes the `using namespace juce` statement. If your Projucer project has this statement disabled, you might want to adjust the CMake project accordingly. To do so, add the following line **before** the `juce_generate_juce_header()` function call:
```cmake
target_compile_definitions("${PROJECT_NAME}" PUBLIC DONT_SET_USING_JUCE_NAMESPACE=1)
```

---

If your project uses binary assets (e.g. images, fonts, audio files), uncomment the following lines and list your asset files:
```cmake
juce_add_binary_data(Assets
    SOURCES <list of asset files separated by spaces>)
```
The file paths can be given relative to the root directory of your project.

`HEADER_NAME` and `NAMESPACE` properties are optional and should remain unset for compatibility with Projucer.

Next, you need to link the `Assets` library to your plugin target. To do so, uncomment `Assets` in the `target_link_libraries()` function call:
```cmake
target_link_libraries("${PROJECT_NAME}"
    PRIVATE
        Assets                # If we'd created a binary data target, we'd link to it here
        juce::juce_audio_utils
    PUBLIC
        juce::juce_recommended_config_flags
        juce::juce_recommended_lto_flags
        juce::juce_recommended_warning_flags)
```

---

If your project requires additional JUCE modules, you can link them in the `PRIVATE` section of the `target_link_libraries()` function call. For example, to link the `juce::juce_dsp` module:
```cmake
target_link_libraries("${PROJECT_NAME}"
    PRIVATE
        Assets                # Here we're assuming that we created a binary data target
        juce::juce_audio_utils
        juce::juce_dsp
    PUBLIC
        juce::juce_recommended_config_flags
        juce::juce_recommended_lto_flags
        juce::juce_recommended_warning_flags)
```

---

### 4. Organise source files into directories

When you create a source file group in the Projucer, a corresponding directory is **not** created in your file system.
All source files are created by the Projucer in the `Source` directory (without subdirectories).

This template requires the source file groups to be represented by subdirectories inside `Source`. Therefore, you need to recreate your Projucer source group organisation with folders:

![SourceDirectories](https://user-images.githubusercontent.com/43878921/155351987-3dd6aa2f-5848-49c1-8441-958bf139914a.png)


If you want to keep using the Projucer project alongside the CMake one, you will have to modify it. All source files that you moved into subdirectories will show up in red in the Projucer. Delete these files from the Projucer, then drag-and-drop the newly created subdirectories into Projucer's "File Explorer" area. Now, some of the `#include` statements in your project will get broken. You will need to prepend the header name with the subdirectory where it is located.

---

### 5. Add `CMakeLists.txt` to source file subdirectories

Create a `CMakeLists.txt` file in the `Source` directory. In this file, you should add:

- all source files that are located directly in the `Source` folder;
- all subdirectories inside `Source` that contain source files.

This is what the file would look like for the example from the [previous step](#4-organise-source-files-into-directories):

```cmake
target_sources("${PROJECT_NAME}"
    PRIVATE
        PluginEditor.h
        PluginEditor.cpp
        PluginProcessor.h
        PluginProcessor.cpp)

add_subdirectory(Components)
add_subdirectory(DSP)
```

- `target_sources()` function is used to add source files to the project target. Header files should be included in the list supplied to the function, or they will not show up in the generated IDE projects.

- `add_subdirectory()` function is used to add directories with source files. Directories added in this way should contain their own `CMakeLists.txt`. We will handle this in the next later in this step.

If you haven't  modified the `#include` statements in [step 4](#4-organise-source-files-into-directories), it is possible to avoid altering your source code altogether. To do so, you need to add the source file subdirectories to the list of include directories for your target. This is done using the `target_include_directories()` function. In our example, you would need to add the following lines to the `CMakeLists.txt` file in the `Source` directory:

```cmake
target_include_directories("${PROJECT_NAME}"
    PRIVATE
        Components
        DSP)
```

---

Now create a `CMakeLists.txt` inside every subdirectory that you added in the previous step. Use the `target_sources()` function to add source files in these subdirectories.

Creating a `CMakeLists.txt` file for every subdirectory makes it easier to rearrange folder organisation. It also helps if you want to copy some source file directory to another project.

---

### 6. Use CMake to generate the build system

The CMake project is now configured. The next step is to generate an IDE project using CMake. To do so, you need to run `cmake` from the root directory of your project, specify the build directory (e.g. `Build` or `Build-CMake`) and the [CMake generator](https://cmake.org/cmake/help/latest/manual/cmake-generators.7.html) to use (e.g. `Xcode` or `"Visual Studio 17 2022"`).

If you want to keep using the Projucer project alongside the CMake one, make sure that the CMake build directory is different from the Projucer one. You could use `Build-CMake` as a CMake build directory for clarity.

---

To generate an **Xcode** project on MacOS, you can run:
```sh
cmake -B Build-CMake -G Xcode -D CMAKE_OSX_ARCHITECTURES=arm64\;x86_64 -D CMAKE_OSX_DEPLOYMENT_TARGET=10.13
```
- `-B Build-CMake` sets the CMake build directory.

- `-G Xcode` tells CMake to generate an Xcode project.

- `-D CMAKE_OSX_ARCHITECTURES=arm64\;x86_64` is an *optional* flag that is required to build universal binaries.

- `-D CMAKE_OSX_DEPLOYMENT_TARGET=10.13` is an *optional* flag that sets the minimum MacOS version to be supported.

---

To generate a **Visual Studio 2022 (17)** on Windows, you can run:
```sh
cmake -B Build-CMake -G "Visual Studio 17 2022"
```
- `-B Build-CMake` sets the CMake build directory.

- `-G "Visual Studio 17 2022"` tells CMake to generate a Visual Studio 2022 project.

---

When you run the appropriate command for your operating system and IDE of choice, CMake will download JUCE into the `Libs` folder and create an IDE project inside the specified build directory.

---

- Every time you add or remove source files, you need to update the appropriate `CMakeLists.txt` files.

- Every time you make any change to any `CMakeLists.txt` file, you need to run CMake to regenerate the IDE project.

To do so, simply run `cmake -B <build directory>`. In our example with `Build-CMake` directory, you would run:
```sh
cmake -B Build-CMake
```
Information about the CMake generator and the optional flags is preserved in the build directory.

---

If you don't intend to keep using the Projucer project, you can delete the following files and folders from the root of your project:
```
Builds/
JuceLibraryCode/
<ProjectName>.jucer
```

---

### 7. "Unused parameter" warnings
After switching from Projucer to CMake, you might encounter "Unused parameter" warnings when building the project.

You can silence these using JUCE's [`juce::ignoreUnused()`](https://docs.juce.com/master/group__juce__core-maths.html#ga5158e6e8faa5553204f90af596fafaf5) function. It accepts any number of arguments and silences the warning for them.

