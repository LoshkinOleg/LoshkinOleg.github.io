# CMakeLists Cheat Sheet

### project("<project_name>")
Names the solution.

### PROJECT_NAME
Variable that contains name of the solution.

### PROJECT_SOURCE_DIR
Variable that holds path to the directory where topmost CMakeLists.txt is located.

### CMAKE_BINARY_DIR
Variable that holds path to the directory where cmake was launched from. Typically is "${PROJECT_SOURCE_DIR}/build/" .

### file(GLOB_RECURSE "<var_name>" "<dir>/*.<extension>")
Makes a list of paths to all files satisfying the wildcard in specified directory and it's sub-directories. Apparently discouraged by modern cmake practices: each file ought to be specified manually.

### add_executable("<exe_name>" "<kwrd>" "<exe_sources>")
Defines a project that builds a single executable. When using target_xxx() functions on it, specify the PRIVATE keyword since nothing should be linking against an executable.

### add_library("<lib_name> <kwrd> <lib_sources>")
Defines a project that builds a single library. Use STATIC as keyword unless you need to build a dll, which case, good luck.

### target_include_directories("<target_name> <kwrd> <sources>")
Gives the specified project access to the specified sources. Use PUBLIC as keyword when the target is a library. Use PRIVATE when the target is an executable since nothing is should be linking against an executable.

### set_target_properties("<target_name> PROPERTIES PREFIX "<prefix>"")
Appends a prefix to a target's output file. Ex: a target named "MyApp" will build a "exe_MyApp.exe" if "exe_" is specified as a prefix.

### Typical order of calls for creating a target:
file(...)
add_xxx(...)
target_xxx(...)

### Make executable for each file in a directory:
file(GLOB_RECURSE executable_files "/main/*.cpp")
foreach(file "${executable_files}")
get_filename_component(executable_name "${file}" NAME_WE)
add_executable("${executable_name}" "${file}")
endforeach()

### add_compile_definitions("<define>")
Add a solution-wide define.

### find_package("<pkg_name> <optional_kwrd> REQUIRED")
Imports a target that can be linked against typically named something like "<pkg_name>::<pkg_name>" .

### add_xxx("<name::name> ALIAS <target>")
Creates an alias for a target to be linked against. Typicaly used by targets you can import with find_package() .

### get_filename_component("<var_name> <path> <params>")
Retrieves string component of a path specified by "<params>".

### Comparing strings:
if ("<str0> STREQUAL <str1>")
...
endif()

### source_group("<name> FILES <sources>")
Creates a visual studio filter containing the files specified.
