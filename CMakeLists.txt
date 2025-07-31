cmake_minimum_required(VERSION 3.10)
project(astra_lua LANGUAGES C)

# 源文件列表
set(CORE_SRC
    lapi.c lcode.c lctype.c ldebug.c ldo.c ldump.c
    lfunc.c lgc.c llex.c lmem.c lobject.c lopcodes.c
    lparser.c lstate.c lstring.c ltable.c ltm.c
    lundump.c lvm.c lzio.c ltests.c
)

set(AUX_SRC
    lauxlib.c
)

set(LIB_SRC
    lbaselib.c ldblib.c liolib.c lmathlib.c loslib.c
    ltablib.c lstrlib.c lutf8lib.c loadlib.c lcorolib.c
    linit.c
)

# 生成静态库
add_library(astra_lua STATIC
    ${CORE_SRC}
    ${AUX_SRC}
    ${LIB_SRC}
)

# 头文件目录
target_include_directories(astra_lua PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/src)

# 跨平台编译选项 - 根据构建类型自动适配
if(MSVC)
    # 通用定义
    target_compile_definitions(astra_lua PRIVATE
        LUA_WIN
        _CRT_SECURE_NO_WARNINGS
    )
    
    # 通用警告设置
    target_compile_options(astra_lua PRIVATE
        /W3
        /wd4244    # 抑制特定警告
        /wd4146
    )
    
    # 根据构建类型设置优化选项
    target_compile_options(astra_lua PRIVATE
        $<$<CONFIG:Debug>:
            /Od        # Debug模式：禁用优化（与/RTC1兼容）
            /Zi        # 生成调试信息
        >
        $<$<CONFIG:Release>:
            /O2        # Release模式：最大化优化
            /Oi        # 启用 intrinsic 函数
            /GL        # 全程序优化
        >
    )
else()
    # GCC/Clang 配置
    set(CWARNSCPP
        -Wfatal-errors
        -Wextra
        -Wshadow
        -Wundef
        -Wwrite-strings
        -Wredundant-decls
        -Wdisabled-optimization
        -Wdouble-promotion
        -Wmissing-declarations
        -Wconversion
        -Wstrict-overflow=2
    )

    set(CWARNGCC
        -Wlogical-op
        -Wno-aggressive-loop-optimizations
    )

    set(CWARNSC
        -Wdeclaration-after-statement
        -Wmissing-prototypes
        -Wnested-externs
        -Wstrict-prototypes
        -Wc++-compat
        -Wold-style-definition
    )

    # 基础编译选项
    target_compile_options(astra_lua PRIVATE
        -Wall
        -fno-stack-protector
        -fno-common
        -march=native
        -std=c99
        ${CWARNSCPP}
        ${CWARNGCC}
        ${CWARNSC}
    )
    
    # 根据构建类型设置优化选项
    target_compile_options(astra_lua PRIVATE
        $<$<CONFIG:Debug>:
            -O0        # Debug模式：禁用优化
            -g         # 生成调试信息
        >
        $<$<CONFIG:Release>:
            -O2        # Release模式：启用优化
        >
    )

    if(UNIX)
        target_compile_definitions(astra_lua PRIVATE LUA_USE_LINUX)
        target_link_libraries(astra_lua PRIVATE -ldl -lm)
    endif()
endif()

# 安装配置
install(TARGETS astra_lua
    ARCHIVE DESTINATION lib
    LIBRARY DESTINATION lib
    RUNTIME DESTINATION bin
)

install(DIRECTORY 
    DESTINATION include/lua
    FILES_MATCHING PATTERN "*.h"
)
    