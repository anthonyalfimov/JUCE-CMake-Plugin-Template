### JUCE CMake Audio Plugin Template ###

cmake_minimum_required(VERSION 3.15)

project(PluginTemplate VERSION 0.0.1)


### Dependency versions ###

set(LIB_JUCE_TAG "6.1.5")


### IDE Generator pre-config ###

# Xcode: Disable automatic build scheme generation globally.
#        Instead, we explicitely specify which targets require schemes.
set(CMAKE_XCODE_GENERATE_SCHEME OFF)

# IDEs:  Enable grouping of source files into folders in IDEs.
set_property(GLOBAL PROPERTY USE_FOLDERS ON)

# IDEs:  Create a folder in the IDE with the JUCE Module code.
option(JUCE_ENABLE_MODULE_SOURCE_GROUPS "Show all module sources in IDE projects" ON)


### Dependency management ###

include(FetchContent)

# Keep dependencies outside of the "Build" directory.
# This allows to do a clean build of the project without re-downloading or
# rebuilding the dependencies.
set(FETCHCONTENT_BASE_DIR "${PROJECT_SOURCE_DIR}/Libs" CACHE PATH "External dependencies path." FORCE)

FetchContent_Declare(juce
    GIT_REPOSITORY https://github.com/juce-framework/JUCE.git
    GIT_TAG        ${LIB_JUCE_TAG}
    GIT_SHALLOW    TRUE
    GIT_CONFIG     advice.detachedHead=false    # Disable detached HEAD warning for fetching a specific tag
    SOURCE_DIR     "${FETCHCONTENT_BASE_DIR}/JUCE"
    SUBBUILD_DIR   "${FETCHCONTENT_BASE_DIR}/JUCE-Subbuild"
    BINARY_DIR     "${FETCHCONTENT_BASE_DIR}/JUCE-Build")

FetchContent_MakeAvailable(juce)


### Plugin Project Setup ###

# If you are building a VST2 or AAX plugin, CMake needs to be told where to find these SDKs.
# This setup should be done before calling `juce_add_plugin`.

# juce_set_vst2_sdk_path(...)
# juce_set_aax_sdk_path(...)

# `juce_add_plugin` adds a static library target with the name passed as the first argument.
# This target is a normal CMake target, but has a lot of extra properties set
# up by default. As well as this shared code static library, this function adds targets for each of
# the formats specified by the FORMATS arguments. This function accepts many optional arguments.
# Check the readme at `docs/CMake API.md` in the JUCE repo for the full list.

juce_add_plugin("${PROJECT_NAME}"
    FORMATS AU VST3                             # The formats to build. Valid formats: Standalone Unity VST3 AU AUv3 AAX VST.
                                                # AU and AUv3 plugins will only be enabled when building on macOS.
    PRODUCT_NAME "Plugin Template"              # The name of the final executable, which can differ from the target name.
    # PLUGIN_NAME                               # Name of the plugin that will be disabled in the DAW. Can differ from PRODUCT_NAME.

    # ICON_BIG                                  # ICON_* arguments specify a path to an image file to use as an icon for the Standalone.
    # ICON_SMALL
    COMPANY_NAME yourcompany                    # The name of this target's author. The value is inherited from JUCE_COMPANY_NAME.
    # COMPANY_WEBSITE                           # The address of a website related to this target in some way.
                                                # The value is inherited from JUCE_COMPANY_WEBSITE.
    # COMPANY_EMAIL                             # An email address for this target's author. The value is inherited from JUCE_COMPANY_EMAIL.
    PLUGIN_MANUFACTURER_CODE Manu               # A four-character manufacturer id with at least one upper-case character.
                                                # GarageBand 10.3 requires the first letter to be upper-case, and the remaining letters to be lower-case.
    PLUGIN_CODE Dem0                            # A unique four-character plugin id with exactly one upper-case character.
                                                # GarageBand 10.3 requires the first letter to be upper-case, and the remaining letters to be lower-case.

    IS_SYNTH FALSE                              # Is this a synth or an effect?
    NEEDS_MIDI_INPUT FALSE                      # Does the plugin need midi input?
    NEEDS_MIDI_OUTPUT FALSE                     # Does the plugin need midi output?
    IS_MIDI_EFFECT FALSE                        # Is this plugin a MIDI effect?
    EDITOR_WANTS_KEYBOARD_FOCUS FALSE           # Does the editor need keyboard focus?

    # VST3_CATEGORIES                           # Should be one or more, separated by spaces, of the following:
                                                # Fx, Instrument, Analyzer, Delay, Distortion, Drum, Dynamics, EQ, External,
                                                # Filter, Generator, Mastering, Modulation, Mono, Network, NoOfflineProcess,
                                                # OnlyOfflineProcess, OnlyRT, Pitch Shift, Restoration, Reverb, Sampler, Spatial,
                                                # Stereo, Surround, Synth, Tools, Up-Downmix
    # AU_MAIN_TYPE                              # Should be one or more, separated by spaces, of the following:
                                                # kAudioUnitType_Effect, kAudioUnitType_FormatConverter, kAudioUnitType_Generator,
                                                # kAudioUnitType_MIDIProcessor, kAudioUnitType_Mixer, kAudioUnitType_MusicDevice,
                                                # kAudioUnitType_MusicEffect, kAudioUnitType_OfflineEffect, kAudioUnitType_Output,
                                                # kAudioUnitType_Panner

    COPY_PLUGIN_AFTER_BUILD FALSE)              # Should the plugin be installed to a default location after building?
                                                # NOTE: Writing to the default install locations might require administrator privileges on Windows.

# Set the C++ language standard requirenment for the "shared code" library target.
# Setting this to PUBLIC ensures that all dependent targets will inherit the specified C++ standard.
target_compile_features("${PROJECT_NAME}" PUBLIC cxx_std_17)

# Disable compiler extensions for the project targets (e.g. use -std=c++17 instead of -std=gnu++17).
get_property(project_targets DIRECTORY "${PROJECT_SOURCE_DIR}" PROPERTY BUILDSYSTEM_TARGETS)

set_target_properties(${project_targets} PROPERTIES CXX_EXTENSIONS OFF)

# `juce_generate_juce_header` will create a JuceHeader.h for a given target, which will be generated
# into your build tree. The include path for this header will be automatically added to the target.

# juce_generate_juce_header("${PROJECT_NAME}")

# Add the subdirectory with source files.
add_subdirectory(Source)

# `target_compile_definitions` adds some preprocessor definitions to our target.
# JUCE modules also make use of compile definitions to switch certain features on/off,
# so if there's a particular feature you need that's not on by default, check the module header
# for the correct flag to set here.
# These definitions will be visible both to your code, and also the JUCE module code.

target_compile_definitions("${PROJECT_NAME}"
    PUBLIC
        # JUCE_WEB_BROWSER and JUCE_USE_CURL would be on by default, but you might not need them.
        JUCE_WEB_BROWSER=0  # If you remove this, add `NEEDS_WEB_BROWSER TRUE` to the `juce_add_plugin` call
        JUCE_USE_CURL=0     # If you remove this, add `NEEDS_CURL TRUE` to the `juce_add_plugin` call
        JUCE_VST3_CAN_REPLACE_VST2=0)

# If your target needs extra binary assets, you can add them here. The first argument is the name of
# a new static library target that will include all the binary resources. There is an optional
# `NAMESPACE` argument that can specify the namespace of the generated binary data class. Finally,
# the SOURCES argument should be followed by a list of source files that should be built into the
# static library. These source files can be of any kind (wav data, images, fonts, icons etc.).
# Conversion to binary-data will happen when your target is built.

# juce_add_binary_data(Assets SOURCES ...)

# `target_link_libraries` links libraries and JUCE modules to other libraries or executables. Here,
# we're linking our executable target to the `juce::juce_audio_utils` module. Inter-module
# dependencies are resolved automatically.

target_link_libraries("${PROJECT_NAME}"
    PRIVATE
        # Assets                # If we'd created a binary data target, we'd link to it here
        juce::juce_audio_utils
    PUBLIC
        juce::juce_recommended_config_flags
        juce::juce_recommended_lto_flags
        juce::juce_recommended_warning_flags)


### IDE Generator post-config ###

# IDEs:  Place source groups for project targets into a "Targets" folder (to reduce visual clutter).
# Xcode: Generate build schemes for project targets.
set_target_properties(${project_targets} PROPERTIES
    FOLDER "Targets"
    XCODE_GENERATE_SCHEME ON)

# IDEs:  Move the "shared code" source group (main target) out of the "Targets" folder
#        to the top level.
set_target_properties("${PROJECT_NAME}" PROPERTIES FOLDER "")

# IDEs:  Generate source groups that follow the organisation of source file directories.
get_target_property(project_sources "${PROJECT_NAME}" SOURCES)

# NOTE:  The following source file grouping does not support JuceHeader.h generation.
#        If JuceHeader.h is required, replace "${PROJECT_SOURCE_DIR}/Source" with "${PROJECT_SOURCE_DIR}".
source_group(TREE "${PROJECT_SOURCE_DIR}/Source" FILES ${project_sources})