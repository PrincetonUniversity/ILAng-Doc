# ILAng with CMake

To utilize ILAng in your C++ projects,  you can look for the `ilang::ilang` interface target in your CMake recipe. This target manages and populates the required usage information such as include directories, linked libraries, compile features, etc. 

### External

With ILAng installed \(out-of-source\), you can locate the target directly with `find_package()` and use the imported target from the generated package configuration:

```text
# CMakeLists.txt
find_package(ilang REQUIRED)

add_library(MyProj ...)

target_link_libraries(MyProj PRIVATE ilang::ilang)
```

### Embedded

ILAng also supports embedded \(in-source\) build. To embed the library into an existing CMake project, place the entire source tree in a sub-directory and call `add_subdirectory()` in your CMake recipe:

```text
# CMakeLists.txt
add_subdirectory(ilang)

add_library(MyProj ...)

target_link_libraries(MyProj PRIVATE ilang::ilang)
```

### Supporting both

To allow your project to support either an externally installed or an embedded library at config time, you can use the following pattern:

```text
# top level CMakeLists.txt
project(MY_PROJ)

option(MY_PROJ_USE_EXTERNAL_ILANG "Use an external ILAng library" OFF)

add_subdirectory(externals)

add_library(MyPorj ...)

target_link_libraries(MyProj PRIVATE ilang::ilang)
```

And put the ILAng source \(or sub-modules\) in the sub-directory `externals/ilang`. 

```text
# externals/CMakeLists.txt

if(MY_PROJ_USE_EXTERNAL_ILANG)
    find_package(ilang REQUIRED)
else()
    add_subdirectory(ilang)
endif()
```

