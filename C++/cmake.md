### With Clion

- when not compiling using clion, "在 Clion 根目录下的 CMakeLists.txt 上右键 Reload Cmake Project"，后方可正确跳转定义等
- 注意需要正确配置 Settings > Build, Execution, Deployment > CMake 下的 Building Directory （build/jiutian）
- 需要 正确选择 Generator （Let CMake decide）
- 其他的第三方依赖如果不能正确跳转，可通过[新功能](https://intellij-support.jetbrains.com/hc/en-us/articles/207253135-CLion-fails-to-find-some-of-my-headers-Where-does-it-search-for-them-) ‘Mark Directory As’ 来标记对应的库 

### Macro

-DYOUR_MACRO_NAME=ON

-DYOUR_MACRO_NAME=some_other_value



disk y00848154 clion 1 4 bin ninja linux x64 ninja



## Conan

包管理器

## GDB

###### request type 

- attach: attach to an already running program
- launch: launch with a debuggable program