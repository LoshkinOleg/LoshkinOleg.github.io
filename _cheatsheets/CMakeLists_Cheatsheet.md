---
title: "CMakeLists Cheatsheet"
layout: cheatsheet
categories: "cheatsheets"
permalink: /:categories/:title
---

# Predefined CMake Functions

<strong>cmake_minimum_required (VERSION 3.14)</strong>
<p style="margin-left:5%;">Define minimal CMake version.</p>

<strong>project(project_name)</strong>
<p style="margin-left:5%;">Names the project / VS solution.</p>

<strong>set (CMAKE_CXX_STANDARD 20)<br>
set (CMAKE_CXX_STANDARD_REQUIRED ON)</strong>
<p style="margin-left:5%;">Sets the minimal C++ standard to use and requires it's usage. NOTE: not all IDE's enforce the usage of the standard! VSCode for instance still needs to be configured manually.</p>

<strong>file(GLOB_RECURSE MyApp_include ${PROJECT_SOURCE_DIR}/MyApp/include/*.h)</strong>
<p style="margin-left:5%;">Makes a list of paths to all files satisfying the wildcard in specified directory and it's sub-directories. Supposedly discouraged by modern cmake practices: each file ought to be specified manually but who in their right mind would bother with that for large repos?</p>

<strong>add_executable(MyApp ${MyApp_include} ${MyApp_src})</strong>
<p style="margin-left:5%;">Defines a project that builds a single executable. When using target_xxx() functions on it, specify the PRIVATE keyword since nothing should be linking against an executable.</p>

<strong>add_library(MyLib STATIC ${MyLib_include} ${MyLib_src})</strong>
<p style="margin-left:5%;">Defines a project that builds a single library. Use STATIC as keyword for static .lib's, SHARED for .dll's and .so's .</p>

<strong>target_sources(MyApp PRIVATE ${someThirdParty_include} ${someThirdParty_src})</strong>
<p style="margin-left:5%;">Add additional sources to an existing target.</p>

<strong>target_include_directories(MyApp PRIVATE<br>
&nbsp;&nbsp;&nbsp;&nbsp;${PROJECT_SOURCE_DIR}/MyLib/include/<br>
&nbsp;&nbsp;&nbsp;&nbsp;${PROJECT_SOURCE_DIR}/MyApp/include/<br>
&nbsp;&nbsp;&nbsp;&nbsp;)</strong>
<p style="margin-left:5%;">Gives the specified project access to the specified source directories. Use PUBLIC as keyword when the target is a library. Use PRIVATE when the target is an executable since nothing is should be linking against an executable.</p>

<strong>target_link_libraries(MyApp PRIVATE<br>
&nbsp;&nbsp;&nbsp;&nbsp;general MyLib<br>
&nbsp;&nbsp;&nbsp;&nbsp;optimized ${PROJECT_SOURCE_DIR}/thirdparty/someLib.lib<br>
&nbsp;&nbsp;&nbsp;&nbsp;debug ${PROJECT_SOURCE_DIR}/thirdparty/someLib_debug.lib<br>
&nbsp;&nbsp;&nbsp;&nbsp;)</strong>
<p style="margin-left:5%;">Links the specified target with other libraries, either defined above or specified as paths. Prepend the dependancies with "general" to specify it as a dependancy for all builds. Prepend dependancies with "optimized" to specify it as a dependancy for Release builds. Prepend dependancies with "debug" to specify it as a dependancy for Debug builds.</p>

<strong>add_compile_definitions(MY_DEFINE ON)</strong>
<p style="margin-left:5%;">Add a solution-wide define.</p>

<strong>source_group(name FILES sources)</strong>
<p style="margin-left:5%;">Creates a visual studio filter containing the files specified.</p>

<strong>add_compile_definitions(APPLICATION_RESOURCES_DIR="${PROJECT_SOURCE_DIR}/resources/")</strong>
<p style="margin-left:5%;">Create a define for a path to be used in the code.</p>

<strong>file(MAKE_DIRECTORY ${PROJECT_SOURCE_DIR}/txtOutputs)</strong>
<p style="margin-left:5%;">Create a directory.</p>

<strong>set_target_properties(Application PROPERTIES RUNTIME_OUTPUT_DIRECTORY "${PROJECT_SOURCE_DIR}/build/MyApp/bin")</strong>
<p style="margin-left:5%;">Define where to put the target's executable. For some reason doesn't work for libs...</p>

<strong>message(WARNING "You're not using Microsoft Visual Studio IDE.")</strong>
<p style="margin-left:5%;">Write a message for the user. STATUS for simple message, FATAL_ERROR to throw error and stop generation. WARNING for warning.</p>

<strong>set_target_properties(MyApp PROPERTIES PREFIX "exe_")</strong>
<p style="margin-left:5%;">Appends a prefix to a target's output file. Ex: a target named "MyApp" will build a "exe_MyApp.exe" if "exe_" is specified as a prefix.</p>

<strong>find_package(pkg_name optional_kwrd REQUIRED)</strong>
<p style="margin-left:5%;">Imports a target that can be linked against typically named something like pkg_name::pkg_name . The optional_kwrd is here if you want to force a particular search mode, leave it empty.</p>

-----
# Predefined CMake Variables

<strong>PROJECT_NAME</strong>
<p style="margin-left:5%;">Variable that contains name of the solution.</p>

<strong>PROJECT_SOURCE_DIR</strong>
<p style="margin-left:5%;">Variable that holds path to the directory where topmost CMakeLists.txt is located.</p>

<strong>CMAKE_BINARY_DIR</strong>
<p style="margin-left:5%;">Variable that holds path to the directory where cmake was launched from. Typically is "${PROJECT_SOURCE_DIR}/build/" .</p>

-----
# Snippets

<strong>file(...)<br>
add_xxx(...)<br>
target_xxx(...)</strong>
<p style="margin-left:5%;">Typical order of calls for creating a target.</p>

<strong>if(NOT EXISTS "${PROJECT_SOURCE_DIR}/build/")<br>
&nbsp;&nbsp;&nbsp;&nbsp;message(FATAL_ERROR "Please specify an out-of-source directory 'build/' in the project's root directory. If you don't know what an out-of-source build is, here's a link: https://cmake.org/cmake/help/book/mastering-cmake/chapter/Getting%20Started.html")<br>
endif()</strong>
<p style="margin-left:5%;">Require the user to make an out-of-source CMake build.</p>

<strong>if (UNIX AND NOT APPLE)<br>
&nbsp;&nbsp;&nbsp;&nbsp;set(LINUX TRUE)<br>
endif()</strong>
<p style="margin-left:5%;">Detect and define linux.</p>

<strong>set(USE_EASY_PROFILER OFF CACHE BOOL "Whether to enable profiling with easy_profiler.")<br>
if (USE_EASY_PROFILER)<br>
&nbsp;&nbsp;&nbsp;&nbsp;add_compile_definitions(BUILD_WITH_EASY_PROFILER)<br>
endif()</strong>
<p style="margin-left:5%;">Create a define that can be toggled on and off in the CMake GUI.</p>

<strong>file(GLOB_RECURSE executable_files "/main/*.cpp")<br>
foreach(file "${executable_files}")<br>
&nbsp;&nbsp;&nbsp;&nbsp;get_filename_component(executable_name "${file}" NAME_WE)<br>
&nbsp;&nbsp;&nbsp;&nbsp;add_executable("${executable_name}" "${file}")<br>
endforeach()</strong>
<p style="margin-left:5%;">Make executable for each file in a directory.</p>

<strong>if ("<str0> STREQUAL <str1>")<br>
&nbsp;&nbsp;&nbsp;&nbsp;doThings...<br>
endif()</strong>
<p style="margin-left:5%;">Comparing strings.</p>
