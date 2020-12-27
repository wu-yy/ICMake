
##  类型

字符串 string

- 基础
    格式 "..."，如 "dd${var}dd"，"a string"

    转义字符加双斜杠，如 "...\\n..."
    
    特殊字符加单斜杠，如 "...\"..."

    ${var} 不同于 "${var}"
 
```c++
set(specialStr "aaa;bbb")

# example 1
message(STATUS ${specialStr}) # aaabbb
message(STATUS "${specialStr}") # aaa;bbb

# example 2
function(PrintVar var)
  message(STATUS "${var}")
endfunction()

PrintVar(${specialStr}) # aaa
PrintVar("${specialStr}") # aaa;bbb
```
- 查找
 
    `string(FIND <string> <substring> <output_variable> [REVERSE])` , 没有找到则结果是-1
 
 - 其他
 
    `string(APPEND <string-var> [<input>...])`
     
     `string(PREPEND <string-var> [<input>...])`
     
     `string(CONCAT <out-var> [<input>...])`
 
     `string(JOIN <glue> <out-var> [<input>...])`
     
     `string(TOLOWER <string> <out-var>)`
     
     `string(TOUPPER <string> <out-var>)`
     
     `string(LENGTH <string> <out-var>)`
     
     `string(SUBSTRING <string> <begin> <length> <out-var>)`
     
     `string(STRIP <string> <out-var>)`
     
     `string(GENEX_STRIP <string> <out-var>)`
     
    ` string(REPEAT <string> <count> <out-var>)`
     
     `string(COMPARE <op> <string1> <string2> <out-var>)`
     
     `string(<HASH> <out-var> <input>)`
     

列表 list

创建:
`set(<list_name> <item>...)` 或者 `set(<list_name> "${item_0};${item_1};...")`

## 调试

- message

`message(STATUS/WARNING/FATAL/ERROR "str")`

## 函数

- 按名传参数 和 按值传参数
```c++
function(PrintVar var)
  message(STATUS "${var}: ${${var}}")
endfunction()

function(PrintValue value)
  message(STATUS "${value}")
endfunction()

set(num 2)
PrintVar(num)

printValue(${num})
```
 
-  cmake_parse_arguments

```
cmake_parse_arguments("ARG" #prefix
    <options> #  TRUE/FALSE
    <multi_value_keyword> #list
    ${ARGN}
)
# 结果为 ARG_*
# - ARG_<option>
# - ARG_<one_value_keyword>
# - ARG_<multi_value_keyword>
```
list 作为参数

    调用时写成 ${list} 会被展开成多个参数
    调用时写成 "${list}"，函数内部参数即为正常 list
    调用时写成 <list>，函数内部需使用 ${arg_list} 得到正常 list


## 控制流

- 循环 n 次

```c++
set(i 0)
while( i LESS <n>)
    # ...
    math(EXPR i "${i} + 1")
endwhile()
```

- 遍历单个list

```c++
foreach(v ${list})
 #...${v};
endforeach()
```

- 遍历两个等长 list

```c++
list(LENGTH <list0> n)
set(i 0)
while(i LESS n)
  list(GET <list0> ${i} v0)
  list(GET <list1> ${i} v1)
  # ...
  math(EXPR i "${i} + 1")
endwhile()
```

- 遍历结构 list

```c
list(LENGTH <struct_list> n)
set(i 0)
while(i LESS n)
  list(SUBLIST <struct_list> ${i} <struct_size> <obj>)
  list(GET <obj> 0 <field_0>)
  list(GET <obj> 1 <field_1>)
  # ...
  list(GET <obj> k <field_k>) # k == <struct_size> - 1
  
  # ...
  math(EXPR i "${i} + ${struct_size}")
endwhile()
```


## 正则表达式

- string

```c
string(REGEX
	MATCH
	<regular_expression>
	<output_variable>
	<input> [<input>...])
# 匹配一次
# CMAKE_MATCH_<n>
# - 由 '()' 句法捕获
# - n : 0, 1, ..., 9
# - CMAKE_MATCH_0 == <output_variable>
# - n == CMAKE_MATCH_COUNT

string(REGEX
	MATCHALL
	<regular_expression>
	<output_variable>
	<input> [<input>...])
# 匹配多次，结果为 list
# CMAKE_MATCH_* 无用

string(REGEX
	REPLACE
	<regular_expression>
	<replacement_expression>
	<output_variable>
	<input> [<input>...])
# \1, \2, ..., \9 表示 '()' 捕获的结果
# 在 <replacement_expression> 中需要写成 \\1, \\2, ..., \\9
```

- list 

```c
list(FILTER <list>
  INCLUDE/EXCLUDE
  REGEX <regular_expression>)
# INCLUDE: 将所有匹配替换成 <list>
# EXCLUDE: 将所有匹配从 <list> 排除
```


## target

- add: [add_executable](https://cmake.org/cmake/help/v3.16/command/add_executable.html), [add_library](https://cmake.org/cmake/help/v3.16/command/add_library.html)

```cmake
#1. add_executable
add_executable(<name> [<source>...])

#2. add_library
# 2.1 normal
add_library(<name> STATIC|SHARED [<source>...])

# 2.2 interface :: pure head files
add_library(<name> INTERFACE)

#3. alias
add_library(<alias> ALIAS <target>)
# <alias> 可以用命名空间 <namespace>::<id>，如 test::XXX
```

- source

```cmake
target_sources(<target>
  PUBLIC    <item>...
  PRIVATE   <item>...
  INTERFACE <item>...
)

# gather sources
# GLOB_RECURSE 获取absolute path
file(GLOB_RECURSE sources
  # header files
  <path>/*.h
  <path>/*.hpp
  <path>/*.hxx
  <path>/*.inl
  
  # source files
  <path>/*.c
  
  <path>/*.cc
  <path>/*.cpp
  <path>/*.cxx
  
  # shader files
  <path>/*.vert # glsl vertex shader
  <path>/*.tesc # glsl tessellation control shader
  <path>/*.tese # glsl tessellation evaluation shader
  <path>/*.geom # glsl geometry shader
  <path>/*.frag # glsl fragment shader
  <path>/*.comp # glsl compute shader
  
  <path>/*.hlsl
  
  # Qt files
  <path>/*.qrc
  <path>/*.ui
)

file(GLOB_RECURSE test_sources include/test.h)

foreach(s ${test_sources})
    MESSAGE(STATUS "test:"${s})
    get_filename_component(dir ${s} DIRECTORY)
    MESSAGE(STATUS "dir:"${dir})
    MESSAGE(STATUS "CMAKE_CURRENT_SOURCE_DIR:"${CMAKE_CURRENT_SOURCE_DIR})
    MESSAGE(STATUS "PROJECT_SOURCE_DIR:"${PROJECT_SOURCE_DIR})
    if(${CMAKE_CURRENT_SOURCE_DIR} STREQUAL ${dir})
        source_group("src" FILES ${s})
    else()
        file(RELATIVE_PATH rdir ${PROJECT_SOURCE_DIR} ${dir})
        MESSAGE(STATUS "rdir2:"${rdir})
        if(MSVC)
            string(REPLACE "/" "\\" rdir_MSVC ${rdir})
            set(rdir "${rdir_MSVC}")
        endif()
        source_group(${rdir} FILES ${s})
    endif()
endforeach()
```

- definition 

为目标增加编译选项
```cmake
target_compile_definitions(<target>
  PUBLIC    <item>...
  PRIVATE   <item>...
  INTERFACE <item>...
)
# <item> => #define <item>
```

- include directories

在编译时会增加 `-I <dir1> -I <dir2>`
```cmake
target_include_directories(mylib PUBLIC
  $<BUILD_INTERFACE:<absolute_path>>
  $<INSTALL_INTERFACE:<relative_path>>  # <install_prefix>/<relative_path>
)
# e.g.
# - <absolute_path>: ${PROJECT_SOURCE_DIR}/include
# - <relative_path>: <package_name>/include
```

- link library

```cmake
target_link_libraries(<target>
  PUBLIC    <item>...
  PRIVATE   <item>...
  INTERFACE <item>...
)
```


## file

- Reading

READ : `file(READ <filename> <out>)`

STRINGS : `file(STRINGS <filename> <variable> [<options>...])`

HAHS : `file(<HASH> <filename> <variable> )`

`<HASH>：MD5/SHA1/SHA224/SHA256/SHA384/SHA512/SHA3_224/SHA3_256/SHA3_384/SHA3_512`

- Writing

WRITE/APPEND: `file(WRITE/APPEND <filename> <content>...)`


TOUCH: `file(TOUCH [<files>...])`

    - 若文件不存在，创建空文件
    - 若文件存在，则更新访问和/或修改时间
    - TOUCH_NOCREATE

GENERATE: `file(GENERATE OUTPUT <output_file> <INPUT input-file|CONTENT content>)`

    - 在 generation 阶段 生成文件
    - 可添加 CONTIDION <expression>，其中 <expression> == 0/1


- file system

GLOB

```cmake
file(GLOB/GLOB_RECURSE <out_list>
     [LIST_DIRECTORIES true|false] # 是否包含目录
     [RELATIVE <path>] # 相对路径
     [CONFIGURE_DEPENDS] # 将结果的所有文件作为 rebuild 的检测对象
     [<globbing-expressions>...] # 简化版的正则表达式
     )
```

```cmake
<globbing-expressions>
ref：Linux Programmer's Manual GLOB
?：匹配单个字符
*：匹配文件名/文件夹名内的任意个字符
**：跨目录匹配任意个字符
[...]：同于正则表达式的 [...]
[!...]：补
```

RENAME : `file(RENAME <oldname> <newname>)` 移动文件或文件夹

REMOVE: `file(REMOVE/REMOVE_RECURSE [<files>...])`
 
 * REMOVE：删除文件，不能删除文件夹
 * REMOVE_RECURSE：删除文件或文件夹
 
MAKE_DIRECTORY：`file(MAKE_DIRECTORY [<directories>...])`，递归创建文件夹

COPY/INSTALL：`file(COPY/INSTALL <files>... DESTINATION <dir>)`，拷贝/安装文件 

- path conversion

```cmake
file(RELATIVE_PATH <variable> <directory> <file>)
file(TO_CMAKE_PATH "<path>" <variable>)，'/'
file(TO_NATIVE_PATH "<path>" <variable>)，Windows '\\'，其他 '/'
```

- transfer
`file(DOWNLOAD <url> <file> [<options>...])`

```cmake
INACTIVITY_TIMEOUT <seconds>：无响应等待时长
TIMEOUT <seconds>：总等待时长
SHOW_PROGReSS：进度
STATUS <variable>：状态为两个值的 list，前者为 0 时表示无错
EXPECTED_HASH <HASH>=<value>：哈希值
```

例子：
```cmake
file(DOWNLOAD
    http://graphics.stanford.edu/pub/3Dscanrep/bunny.tar.gz
    ${CMAKE_BINARY_DIR}/demo/bunny.tar.gz
    SHA512=59e7b43db838dbe6f02fda5e844d18e190d32d2ca1a83dc9f6b1aaed43e0340fc1e8ecabed6fffdac9f85bd34e7e93b5d8a17283d59ea3c14977a0372785d2bd
    SHOW_PROGRESS
)
add_custom_target(demo tar -xzf ${CMAKE_BINARY_DIR}/demo/bunny.tar.gz -C ${CMAKE_BINARY_DIR}/demo)
```

## package

- `<namespace>`，e.g. test

- `<package_name>`

```cmake
e.g. ${PROJECT_NAME}_${PROJECT_VERSION_MAJOR}_${PROJECT_VERSION_MINOR}_${PROJECT_VERSION_PATCH}

```

- target name：`${PROJECT_NAME}_${relative_path}`，其中 / 要转成 _

`string(REPLACE "/" "_" targetName "${PROJECT_NAME}_${relative_path}")`


- bin, dll, lib path
```cmake
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${PROJECT_SOURCE_DIR}/bin")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_DEBUG "${PROJECT_SOURCE_DIR}/bin")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_RELEASE "${PROJECT_SOURCE_DIR}/bin")
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${PROJECT_SOURCE_DIR}/lib")
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY_DEBUG "${PROJECT_SOURCE_DIR}/lib")
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY_RELEASE "${PROJECT_SOURCE_DIR}/lib")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${PROJECT_SOURCE_DIR}/lib")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY_DEBUG "${PROJECT_SOURCE_DIR}/lib")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY_RELEASE "${PROJECT_SOURCE_DIR}/lib")
```

- debug postfix

dll, lib: `set(CMAKE_DEBUG_POSTFIX d)`

exe:  `set_target_properties(<target> PROPERTIES DEBUG_POSTFIX ${CMAKE_DEBUG_POSTFIX})`

- install

```
install(TARGETS <target>...
  EXPORT "${PROJECT_NAME}Targets" # 链接 export
  RUNTIME DESTINATION bin # .exe, .dll
  LIBRARY DESTINATION "${package_name}/lib" # non-DLL shared library
  ARCHIVE DESTINATION "${package_name}/lib" # .lib
)

install(FILES "${PROJECT_SOURCE_DIR}/include" DESTINATION "${package_name}/include"
```

- install export
```cmake
install(EXPORT "${PROJECT_NAME}Targets"
  NAMESPACE <namespace>
)
```

- config

```cmake
include(CMakePackageConfigHelpers)

configure_package_config_file(${PROJECT_SOURCE_DIR}/config/Config.cmake.in
  "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
  INSTALL_DESTINATION "${package_name}/cmake" # 仅生成文件使用，后续还需要自行 install
  NO_SET_AND_CHECK_MACRO
  NO_CHECK_REQUIRED_COMPONENTS_MACRO
  PATH_VARS "${package_name}/include"
)

write_basic_package_version_file(
  "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
  VERSION ${PROJECT_VERSION}
  COMPATIBILITY ExactVersion
)

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake
              ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
        DESTINATION "${PROJECT_NAME}/cmake"
)
```