cmake_minimum_required(VERSION 3.15)

project(
    luajit
    VERSION 2.1
    DESCRIPTION "LuaJit build script created by Jakit"
    HOMEPAGE_URL "https://github.com/jakitliang/LuaJIT"
)

option(LUAJIT_OPTION_DISABLE_FFI "Permanently disable the FFI extension" OFF)
option(LUAJIT_OPTION_ENABLE_LUA52COMPAT "Enable partial 5.2 features" OFF)
option(LUAJIT_OPTION_DISABLE_JIT "Disable the JIT compiler" OFF)

if (CMAKE_SYSTEM_NAME MATCHES "Windows")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} /nologo /c /D_CRT_SECURE_NO_DEPRECATE /D_CRT_STDIO_INLINE=__declspec(dllexport)__inline")

    set(DASM "${CMAKE_CURRENT_SOURCE_DIR}/dynasm/dynasm.lua")
    if (CMAKE_SIZEOF_VOID_P EQUAL 8)
        # 64 bits
        if (CMAKE_SYSTEM_PROCESSOR MATCHES "AMD64")
            # amd64
            set(DASMFLAGS -LN -D WIN -D JIT -D FFI -D ENDIAN_LE -D FPU -D P64)
            set(DASC "${CMAKE_CURRENT_SOURCE_DIR}/src/vm_x64.dasc")
        else ()
            # arm
            set(DASMFLAGS -D WIN -D JIT -D FFI -D ENDIAN_LE -D FPU)
            set(DASC "${CMAKE_CURRENT_SOURCE_DIR}/src/vm_arm64.dasc")
        endif ()

    elseif (CMAKE_SIZEOF_VOID_P EQUAL 4)
        # 32 bits
        # x86
        set(DASMFLAGS -D WIN -D JIT -D FFI -D ENDIAN_LE -D FPU)
        set(DASC "${CMAKE_CURRENT_SOURCE_DIR}/src/vm_x86.dasc")
        set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} /arch:SSE2")
    endif ()

    add_subdirectory(src)
else ()
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fomit-frame-pointer")

    set(DASM "${CMAKE_CURRENT_SOURCE_DIR}/dynasm/dynasm.lua")

    if (CMAKE_SIZEOF_VOID_P EQUAL 8)
        # 64 bits
        if (CMAKE_SYSTEM_PROCESSOR MATCHES "AMD64" OR CMAKE_SYSTEM_PROCESSOR MATCHES "x86_64")
            # amd64
            set(DASMFLAGS -LN -D JIT -D FFI -D ENDIAN_LE -D FPU -D P64)
            set(DASC "${CMAKE_CURRENT_SOURCE_DIR}/src/vm_x64.dasc")
        else ()
            # arm
            set(DASMFLAGS -D JIT -D FFI -D ENDIAN_LE -D FPU)
            set(DASC "${CMAKE_CURRENT_SOURCE_DIR}/src/vm_arm64.dasc")
        endif ()

    elseif (CMAKE_SIZEOF_VOID_P EQUAL 4)
        # 32 bits
        # x86
        set(DASMFLAGS -D JIT -D FFI -D ENDIAN_LE -D FPU)
        set(DASC "${CMAKE_CURRENT_SOURCE_DIR}/src/vm_x86.dasc")
        set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} /arch:SSE2")
    endif ()

    add_subdirectory(src)
endif ()
