# cmake 实践

- `message(STATUS "...")` 调试信息
- 用`${}`来引用变量
- 如在`IF`控制语句，变量是直接使用变量名引用，而不需要`${}`

## 基本语法规则

1. 变量使用`${}`方式取值，但是在`IF`控制语句中是直接使用变量名
1. 指令(参数1 参数2...)
    参数使用括弧括起，参数之间使用空格或分号分开。
1. 指令是大小写无关的，参数和变量是大小写相关的。但，`推荐全部使用大写指令`。

## PROJECT 指令

```
PROJECT(projectname [CXX] [C] [Java])

定义工程名称，并可指定工程支持的语言，支持的语言列表是可以忽略的， 默认情况表示支持所有语言。
该指令隐式的定义了两个 cmake 变量: <projectname>_BINARY_DIR 以及 <projectname>_SOURCE_DIR.
同时预定义了 PROJECT_BINARY_DIR 和 PROJECT_SOURCE_DIR 变量.
建议直接使用 PROJECT_BINARY_DIR，PROJECT_SOURCE_DIR，即使修改了工程名称，也不会影响这两个变量。
```

## SET 指令

```
SET(VAR [VALUE] [CACHE TYPE DOCSTRING [FORCE]])
```

## MESSAGE 指令

```
MESSAGE([SEND_ERROR | STATUS | FATAL_ERROR] "message to display"...)

SEND_ERROR，产生错误，生成过程被跳过。
SATUS，输出前缀为—的信息。
FATAL_ERROR，立即终止所有 cmake 过程.
```

## `ADD_EXECUTABLE`

## `ADD_SUBDIRECTORY` 指令

```
ADD_SUBDIRECTORY(source_dir [binary_dir] [EXCLUDE_FROM_ALL])
用于向当前工程添加存放源文件的子目录，并可以指定中间二进制和目标二进制存放的位置。EXCLUDE_FROM_ALL 参数的含义是将这个目录从编译过程中排除.
```

可以通过`SET`指令重新定义`EXECUTABLE_OUTPUT_PATH`和`LIBRARY_OUTPUT_PATH`变量来指定最终的目标二进制的位置.

```
SET(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin)
SET(LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/lib)

在哪里 ADD_EXECUTABLE 或 ADD_LIBRARY， 如果需要改变目标存放路径，就在哪里加入上述的定义。
```

## 安装到哪?

`cmake -DCMAKE_INSTALL_PREFIX=/usr`

## `INSTALL` 指令

```
INSTALL(TARGETS targets...
      [[ARCHIVE|LIBRARY|RUNTIME]
                    [DESTINATION <dir>]
                    [PERMISSIONS permissions...]
                    [CONFIGURATIONS
      [Debug|Release|...]]
                    [COMPONENT <component>]
                    [OPTIONAL]
                   ] [...])

TARGETS 后面跟的就是通过 ADD_EXECUTABLE 或者 ADD_LIBRARY 定义的 目标文件，可能是可执行二进制、动态库、静态库。
目标类型也就相对应的有三种，ARCHIVE 特指静态库，LIBRARY 特指动态库，RUNTIME 特指可执行目标二进制。
DESTINATION 定义了安装的路径，如果路径以/开头，那么指的是绝对路径，这时候 CMAKE_INSTALL_PREFIX 其实就无效了。如果希望使用 CMAKE_INSTALL_PREFIX 来定义安装路径，就要写成相对路径，即不要以/开头，那么安装后的路径就是 ${CMAKE_INSTALL_PREFIX}/<DESTINATION 定义的路径
```

```
INSTALL(FILES files... DESTINATION <dir>
        [PERMISSIONS permissions...]
        [CONFIGURATIONS [Debug|Release|...]]
        [COMPONENT <component>]
        [RENAME <name>] [OPTIONAL])

安装一般文件，并可以指定访问权限，文件名是此指令所在路径下的相对路径。如果默认不定义权限 PERMISSIONS，安装后的权限为: OWNER_WRITE, OWNER_READ, GROUP_READ,和WORLD_READ，即644权限。
```

```
非目标文件的可执行程序安装

INSTALL(PROGRAMS files... DESTINATION <dir>
        [PERMISSIONS permissions...]
        [CONFIGURATIONS [Debug|Release|...]]
        [COMPONENT <component>]
        [RENAME <name>] [OPTIONAL])

与 FILES 指令使用方法一样，唯一的不同是安装后权限为: OWNER_EXECUTE, GROUP_EXECUTE, 和WORLD_EXECUTE，即755权限
```

```
目录的安装

INSTALL(DIRECTORY dirs... DESTINATION <dir>
        [FILE_PERMISSIONS permissions...]
        [DIRECTORY_PERMISSIONS permissions...]
        [USE_SOURCE_PERMISSIONS]
        [CONFIGURATIONS [Debug|Release|...]]
        [COMPONENT <component>]
        [[PATTERN <pattern> | REGEX <regex>]
         [EXCLUDE] [PERMISSIONS permissions...]] [...])

DIRECTORY 后面连接的是所在 Source 目录的相对路径
如果目录名不以 / 结尾，那么这个目录将被安装为目标路径下的目录，如果目录名以/结尾，代表将这个目录中的内容安装到目标路径，但不包括这个目录本身。
PATTERN 用于使用正则表达式进行过滤，PERMISSIONS 用于指定 PATTERN 过滤后的文件权限。
```

## `ADD_LIBRARY` 指令

```
ADD_LIBRARY(libname    [SHARED|STATIC|MODULE]
      [EXCLUDE_FROM_ALL]
            source1 source2 ... sourceN)

类型有三种:
SHARED，动态库
STATIC，静态库
MODULE，在使用 dyld 的系统有效，如果不支持 dyld，则被当作 SHARED 对待。

EXCLUDE_FROM_ALL 参数的意思是这个库不会被默认构建，除非有其他的组件依赖或者手工构建。
```

## `SET_TARGET_PROPERTIES` 指令

```
SET_TARGET_PROPERTIES(target1 target2 ...
      PROPERTIES prop1 value1
      prop2 value2 ...)

用来设置输出的名称，对于动态库，还可以用来指定动态库版本和 API 版本。
```

## `INCLUDE_DIRECTORIES` 指令

```
INCLUDE_DIRECTORIES([AFTER|BEFORE] [SYSTEM] dir1 dir2 ...)

向工程添加多个特定的头文件搜索路径，路径之间用空格分割
```

1. `CMAKE_INCLUDE_DIRECTORIES_BEFORE`，通过`SET`这个`cmake`变量为`on`，可以将添加的头文件搜索路径放在已有路径的前面。
2. 通过`AFTER`或者`BEFORE`参数，也可以控制是追加还是置前。

## 添加共享库

```
LINK_DIRECTORIES 和 TARGET_LINK_LIBRARIES

LINK_DIRECTORIES(directory1 directory2 ...)

TARGET_LINK_LIBRARIES(target library1
        <debug | optimized> library2
        ...)
```

## 特殊的环境变量 `CMAKE_INCLUDE_PATH` 和 `CMAKE_LIBRARY_PATH`

务必注意，这两个是环境变量而不是 cmake 变量。

## cmake 常用变量

- `PROJECT_BINARY_DIR` 
    如果是`in source`编译，指得就是工程顶层目录，如果是`out-of-source`编译，指的是工程编译发生的目录。
- `CMAKE_SOURCE_DIR` `PROJECT_SOURCE_DIR` 工程顶层目录
- `CMAKE_CURRENT_SOURCE_DIR` 指的是当前处理的`CMakeLists.txt`所在的路径
- `CMAKE_CURRRENT_BINARY_DIR` 如果是`in-source`编译，它跟`CMAKE_CURRENT_SOURCE_DIR`一致，如果是`out-of-source` 编译，他指的是`target`编译目录。
- `CMAKE_CURRENT_LIST_FILE` 输出调用这个变量的`CMakeLists.txt`的完整路径
- `CMAKE_CURRENT_LIST_LINE` 输出这个变量所在的行
- `CMAKE_MODULE_PATH` 用来定义自己的 cmake 模块所在的路径。
- `EXECUTABLE_OUTPUT_PATH` 和 `LIBRARY_OUTPUT_PATH` 分别用来重新定义最终结果的存放目录
- `PROJECT_NAME` 返回通过 PROJECT 指令定义的项目名称。

## cmake 调用环境变量的方式

```
使用 $ENV{NAME} 指令调用系统的环境变量
设置环境变量的方式是: SET(ENV{变量名} 值)
```

1. `CMAKE_INCLUDE_CURRENT_DIR`
```
自动添加 CMAKE_CURRENT_BINARY_DIR 和 CMAKE_CURRENT_SOURCE_DIR 到当前处理的 CMakeLists.txt。相当于在每个 CMakeLists.txt 加入:
INCLUDE_DIRECTORIES(${CMAKE_CURRENT_BINARY_DIR}
${CMAKE_CURRENT_SOURCE_DIR})
```
2. `CMAKE_INCLUDE_DIRECTORIES_PROJECT_BEFORE` 将工程提供的头文件目录始终至于系统头文件目录的前面
3. `CMAKE_INCLUDE_PATH` 和 `CMAKE_LIBRARY_PATH`

## 系统信息

1. `CMAKE_MAJOR_VERSION`，`CMAKE` 主版本号，比如 2.4.6 中的 2
2. `CMAKE_MINOR_VERSION`，`CMAKE` 次版本号，比如 2.4.6 中的 4
3. `CMAKE_PATCH_VERSION`，`CMAKE`补丁等级，比如2.4.6 中的6
4. `CMAKE_SYSTEM`，系统名称，比如 Linux-2.6.22
5. `CMAKE_SYSTEM_NAME`，不包含版本的系统名，比如 Linux
6. `CMAKE_SYSTEM_VERSION`，系统版本，比如 2.6.22
7. `CMAKE_SYSTEM_PROCESSOR`，处理器名称，比如 i686.
8. `UNIX`，在所有的类`UNIX`平台为`TRUE`，包括`OS X`和`cygwin`
9. `WIN32`，在所有的`win32`平台为`TRUE`，包括`cygwin`

## 主要的开关选项

1. `CMAKE_ALLOW_LOOSE_LOOP_CONSTRUCTS`，用来控制`IF ELSE`语句的书写方式
2. `BUILD_SHARED_LIBS` 用来控制默认的库编译方式，如果不进行设置，使用`ADD_LIBRARY`并没有指定库类型的情况下，默认编译生成的库都是静态库。如果`SET(BUILD_SHARED_LIBS ON)`后，默认生成的为动态库。
3. `CMAKE_C_FLAGS` 设置`C`编译选项，也可以通过指令`ADD_DEFINITIONS()`添加。
4. `CMAKE_CXX_FLAGS` 设置`C++`编译选项，也可以通过指令`ADD_DEFINITIONS()`添加。

## `ADD_DEFINITIONS` 向`C/C++`编译器添加`-D`定义

```
ADD_DEFINITIONS(-DENABLE_DEBUG -DABC)，参数之间用空格分割。

如果代码中定义了
#ifdef ENABLE_DEBUG
#endif
这块代码块就会生效。

如果要添加其他的编译器开关，可以通过 CMAKE_C_FLAGS 变量和 CMAKE_CXX_FLAGS 变量设置。
```

## `ADD_DEPENDENCIES`

```
定义 target 依赖的其他 target，确保在编译本 target 之前，其他的 target 已经被构建。

ADD_DEPENDENCIES(target-name depend-target1
     depend-target2 ...)
```

## `AUX_SOURCE_DIRECTORY`

```
AUX_SOURCE_DIRECTORY(dir VARIABLE)

作用是发现一个目录下所有的源代码文件并将列表存储在一个变量中，这个指令临时被用来自动构建源文件列表。因为目前 cmake 还不能自动发现新添加的源文件。

示例:
AUX_SOURCE_DIRECTORY(. SRC_LIST)
ADD_EXECUTABLE(main ${SRC_LIST})
```

## `CMAKE_MINIMUM_REQUIRED`

```
CMAKE_MINIMUM_REQUIRED(VERSION versionNumber [FATAL_ERROR])
```

## `FILE` 指令

```
FILE(WRITE filename "message to write"... )
FILE(APPEND filename "message to write"... )
FILE(READ filename variable)
FILE(GLOB  variable [RELATIVE path] [globbing expressions]...)
FILE(GLOB_RECURSE variable [RELATIVE path] [globbing expressions]...)
FILE(REMOVE [directory]...)
FILE(REMOVE_RECURSE [directory]...)
FILE(MAKE_DIRECTORY [directory]...)
FILE(RELATIVE_PATH variable directory file)
FILE(TO_CMAKE_PATH path result)
FILE(TO_NATIVE_PATH path result)
```

## `FIND_指令`

```
FIND_FILE(<VAR> name1 path1 path2 ...)      VAR 变量代表找到的文件全路径，包含文件名
FIND_LIBRARY(<VAR> name1 path1 path2 ...)   VAR 变量表示找到的库全路径，包含库文件名
FIND_PATH(<VAR> name1 path1 path2 ...)      VAR 变量代表包含这个文件的路径。
FIND_PROGRAM(<VAR> name1 path1 path2 ...)   VAR 变量代表包含这个程序的全路径。

FIND_PACKAGE(<name> [major.minor] [QUIET] [NO_MODULE]
     [[REQUIRED|COMPONENTS] [componets...]])
```

## `IF`指令

```
IF(expression)
    # THEN section.
    COMMAND1(ARGS ...)
    COMMAND2(ARGS ...)
    ...
ELSE(expression)
    # ELSE section.
    COMMAND1(ARGS ...)
    COMMAND2(ARGS ...)
    ...
ENDIF(expression)

另外一个指令是 ELSEIF，总体把握一个原则，凡是出现 IF 的地方一定要有对应的
ENDIF.出现 ELSEIF 的地方，ENDIF 是可选的。

IF(var)，如果变量不是:空，0，N, NO, OFF, FALSE, NOTFOUND 或 <var>_NOTFOUND 时，表达式为真。
IF(NOT var )，与上述条件相反。
IF(var1 AND var2)，当两个变量都为真是为真。
IF(var1 OR var2)，当两个变量其中一个为真时为真。
IF(COMMAND cmd)，当给定的cmd确实是命令并可以调用是为真。
IF(EXISTS dir)或者IF(EXISTS file)，当目录名或者文件名存在时为真。
IF(file1 IS_NEWER_THAN file2)，当 file1 比 file2 新，或者 file1/file2 其中有一个不存在时为真，文件名请使用完整路径。
IF(file1 IS_NEWER_THAN file2)，当 file1 比 file2 新，或者 file1/file2 其 中有一个不存在时为真，文件名请使用完整路径。

IF(variable MATCHES regex)
IF(string MATCHES regex)
当给定的变量或者字符串能够匹配正则表达式 regex 时为真。

数字比较表达式
IF(variable LESS number)
IF(string LESS number)
IF(variable GREATER number)
IF(string GREATER number)
IF(variable EQUAL number)
IF(string EQUAL number)

按照字母序的排列进行比较
IF(variable STRLESS string)
IF(string STRLESS string)
IF(variable STRGREATER string)
IF(string STRGREATER string)
IF(variable STREQUAL string)
IF(string STREQUAL string)

IF(DEFINED variable)，如果变量被定义，为真。
```

```
SET(CMAKE_ALLOW_LOOSE_LOOP_CONSTRUCTS ON)

IF(WIN32)
    ...
ELSE()
    ...
ENDIF()
```

```
IF(WIN32)
    #do something related to WIN32 ELSEIF(UNIX)
    #do something related to UNIX
ELSEIF(APPLE)
    #do something related to APPLE
ENDIF(WIN32)
```

