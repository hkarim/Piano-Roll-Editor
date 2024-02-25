# cmake -DCMAKE_TOOLCHAIN_FILE=$VCPKG_ROOT/scripts/buildsystems/vcpkg.cmake . -B cmake-build
# cmake --build cmake-build --target help
# cmake --build cmake-build --config Release --target PianoRoll

cmake_minimum_required(VERSION 3.15)

project(PianoRoll VERSION 0.0.1)

# on macOS Sonoma and LLVM Clang 17.0.6, CMake invokes `ar` and `ranlib` under `/usr/bin` which causes corrupt libs.
# force the use of `llvm-ar` and `llvm-ranlib` if we are using LLVM
# export LLVM=$(brew --prefix llvm)
if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
    message("LLVM : $ENV{LLVM}")
    set(CMAKE_AR "$ENV{LLVM}/bin/llvm-ar" CACHE FILEPATH "LLVM ar" FORCE)
    set(CMAKE_RANLIB "$ENV{LLVM}/bin/llvm-ranlib" CACHE FILEPATH "LLVM ranlib" FORCE)
    message("AR : ${CMAKE_AR}")
    message("RANLIB : ${CMAKE_RANLIB}")
endif ()

#set(CMAKE_VERBOSE_MAKEFILE ON)
set(CMAKE_CXX_STANDARD 20)
# disable some JUCE objective-c code warnings
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-receiver-forward-class -Wno-objc-method-access")

# uncomment the following line if you have checked out JUCE locally in the current directory
#add_subdirectory(JUCE)

# use the following if you've installed JUCE globally or using a package manager, `vcpkg` for example
find_package(JUCE CONFIG REQUIRED)


juce_add_gui_app(PianoRoll
    PRODUCT_NAME "PianoRoll"
) 

juce_generate_juce_header(PianoRoll)

target_sources(PianoRoll
    PRIVATE
    Source/KeyboardComponent.cpp
    Source/MainComponent.cpp
    Source/NoteGridControlPanel.cpp
    Source/NoteModel.cpp
    Source/PianoRollEditorComponent.cpp
    Source/Main.cpp
    Source/NoteGridComponent.cpp
    Source/NoteGridStyleSheet.cpp
    Source/PNoteComponent.cpp
    Source/TimelineComponent.cpp
)

target_compile_definitions(PianoRoll
    PRIVATE
        # JUCE_WEB_BROWSER and JUCE_USE_CURL would be on by default, but you might not need them.
        JUCE_WEB_BROWSER=0  # If you remove this, add `NEEDS_WEB_BROWSER TRUE` to the `juce_add_gui_app` call
        JUCE_USE_CURL=0     # If you remove this, add `NEEDS_CURL TRUE` to the `juce_add_gui_app` call
        JUCE_APPLICATION_NAME_STRING="$<TARGET_PROPERTY:PianoRoll,JUCE_PRODUCT_NAME>"
        JUCE_APPLICATION_VERSION_STRING="$<TARGET_PROPERTY:PianoRoll,JUCE_VERSION>")

target_link_libraries(PianoRoll
    PRIVATE
        juce::juce_gui_extra
        juce::juce_audio_utils
    PUBLIC
        juce::juce_recommended_config_flags
        juce::juce_recommended_lto_flags
        juce::juce_recommended_warning_flags)
