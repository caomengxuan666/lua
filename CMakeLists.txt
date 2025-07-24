cmake_minimum_required(VERSION 3.10)
project(astra_lua LANGUAGES C)  # 项目名也同步修改

# 源文件列表（保持不变）
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

# 生成静态库，目标名改为 astra_lua
add_library(astra_lua STATIC
    ${CORE_SRC}
    ${AUX_SRC}
    ${LIB_SRC}
)

# 头文件目录（保持不变）
target_include_directories(astra_lua PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/src)

# 跨平台编译选项（保持不变，仅将目标名改为 astra_lua）
if(MSVC)
    target_compile_definitions(astra_lua PRIVATE
        LUA_WIN
        _CRT_SECURE_NO_WARNINGS
    )
    target_compile_options(astra_lua PRIVATE
        /W3
        /O2
        /wd4244
        /wd4146
    )
else()
    # GCC/Clang 配置（保持不变，目标名改为 astra_lua）
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

    target_compile_options(astra_lua PRIVATE
        -Wall
        -O2
        -fno-stack-protector
        -fno-common
        -march=native
        -std=c99
        ${CWARNSCPP}
        ${CWARNGCC}
        ${CWARNSC}
    )

    if(UNIX)
        target_compile_definitions(astra_lua PRIVATE LUA_USE_LINUX)
        target_link_libraries(astra_lua PRIVATE -ldl -lm)
    endif()
endif()

# 安装配置（目标名改为 astra_lua）
install(TARGETS astra_lua
    ARCHIVE DESTINATION lib
    LIBRARY DESTINATION lib
    RUNTIME DESTINATION bin
)

install(DIRECTORY 
    DESTINATION include/lua
    FILES_MATCHING PATTERN "*.h"
)
