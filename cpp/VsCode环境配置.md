# 插件
- C/C++ Extension Pack
- CMake,CMake-Tools
- clang-format
- clangd

## CMake,CMake-Tools
工程目录下新建`CMakeLists.txt`
```
cmake_minimum_required(VERSION 3.22.1)
project(B+TREE-PROJECT)

add_subdirectory(src)
```

src目录下也新建`CMakeLists.txt`
```
add_library(B_Plus_Tree STATIC B_Plus_Tree.cpp)

target_include_directories(B_Plus_Tree PRIVATE ${CMAKE_SOURCE_DIR}/include)

add_executable(main Main.cpp)

target_link_libraries(main PRIVATE B_Plus_Tree)

target_include_directories(main PRIVATE ${CMAKE_SOURCE_DIR}/include)

```

### 使用Cmake

1. 工程目录下新建一个`build`,在`build`目录下输入命令
```
cmake ..
```
2. 或：工程目录下直接输入
```
cmake -S . -B ./build

以后重新构建：
cmake --build ./build
```

### cmake-tools
`c_cpp_properties.json`中添加
```
"configurationProvider": "ms-vscode.cmake-tools"
```
`ctrl+shift+P`，输入`cmake`找到设置，选一个编译器。点击后在侧栏`CMake`大纲就有东西了

### 使用`compile_commands.json`提供跳转功能
#### 生成
- 利用`make`生成：使用bear
```
bear -- make
```
- 利用`CMake`生成：
在工程目录下的`CMakeLists.txt`中添加
```
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
```
#### 使用
`c_cpp_properties.json`中添加
```
"compileCommands":"${workspaceFolder}/build/compile_commands.json"
```

## clangd
工程目录下创建`.clangd`
```
CompileFlags:
    CompilationDatabase: ./build
```

## Clang-Format
格式化工具

## valgrind
```
valgrind ./main
```

## gprof

```
# gprof
SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -pg")
SET(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -pg")
SET(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -pg")
```
执行后会生成`gmon.out`
```
结果到txt文件中:
gprof gmon.out > ./doc/gprofresult.txt
```
画图:
- 安装`gprof2dot`:
```
pip install gprof2dot

gprof myprogram gmon.out>output.txt
gprof2dot output.txt>output.dot
dot -Tpng output.dot -o output.png
//上述三步可以合成一条命令
gprof myprogram gmon.out | gprof2dot | dot -Tpng -o output.png
```

## perf
