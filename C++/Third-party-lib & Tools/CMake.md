# Built-in Variable

- `CMAKE_BINARY_DIR`: top level dir of the CMake build tree, specified by `-B` argument 
- `CMAKE_SOURCE_DIR`: top level dir of the CMake source tree, specified by `-S` argument
- `CMAKE_CURRENT_LIST_DIR`: 
- `CMAKE_INSTALL_LIBDIR`: deside the post-fix of library install path e.g. `lib` or `lib64`



# Key Commands

- `add_library`: define a target library with flag (shared/static) and source files
- `add_executable`: define a target exetutable with source files
- `target_include_directories`: add dependent header files for target
- `target_link_libraries`: add dependent libraries for target
- `aux_source_directory`: add source file directories to a variable
- `include_directories`: Add the given directories for compiler to search for include files. 
- `link_directories`: Add directories in which the linker will look for libraries.
- `add_subdirectory`: a subdirectory to the build(has its own CMakeLists.txt)
- `include`: Loads and runs CMake code from the file given
- `message`: print desired info/variables
- `find_package`: 
  - Module mode: search for a `Find<PackageName>.cmake` file
  - Config mode: search for a `<PackageName>Config.cmake` or `<lowercasePackageName>-config.cmake` file
  - FetchContent redirection mode: redirect to a package provided by`FetchContent` module




# 小知识

1. CMake 编译出的二进制文件不需要配置环境变量 `LD_LIBRARY_PATH` 就可以自动找到依赖库，这是因为 CMake 在编译过程中已经将依赖库的鹿筋写入到可执行文件的rpath中了



### With Clion

- when not compiling using clion, "在 Clion 根目录下的 CMakeLists.txt 上右键 Reload Cmake Project"，后方可正确跳转定义等
- 注意需要正确配置 Settings > Build, Execution, Deployment > CMake 下的 Building Directory （build/jiutian）
- 需要 正确选择 Generator （Let CMake decide）
- 其他的第三方依赖如果不能正确跳转，可通过[新功能](https://intellij-support.jetbrains.com/hc/en-us/articles/207253135-CLion-fails-to-find-some-of-my-headers-Where-does-it-search-for-them-) ‘Mark Directory As’ 来标记对应的库 

### Macro

-DYOUR_MACRO_NAME=ON

-DYOUR_MACRO_NAME=some_other_value



## Conan

C++跨平台包管理器



## GDB

###### request type 

- attach: attach to an already running program
- launch: launch with a debuggable program