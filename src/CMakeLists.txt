
list(
        APPEND
        ALL_LIB
        ${CMAKE_CURRENT_SOURCE_DIR}/lib_base.c
        ${CMAKE_CURRENT_SOURCE_DIR}/lib_math.c
        ${CMAKE_CURRENT_SOURCE_DIR}/lib_bit.c
        ${CMAKE_CURRENT_SOURCE_DIR}/lib_string.c
        ${CMAKE_CURRENT_SOURCE_DIR}/lib_table.c
        ${CMAKE_CURRENT_SOURCE_DIR}/lib_io.c
        ${CMAKE_CURRENT_SOURCE_DIR}/lib_os.c
        ${CMAKE_CURRENT_SOURCE_DIR}/lib_package.c
        ${CMAKE_CURRENT_SOURCE_DIR}/lib_debug.c
        ${CMAKE_CURRENT_SOURCE_DIR}/lib_jit.c
        ${CMAKE_CURRENT_SOURCE_DIR}/lib_ffi.c
        ${CMAKE_CURRENT_SOURCE_DIR}/lib_buffer.c
)

set(MiniLua ${CMAKE_BINARY_DIR}/src/host/minilua.exe)
set(BuildVM ${CMAKE_BINARY_DIR}/src/host/buildvm.exe)

add_subdirectory(host)

add_library(
        lua51
        SHARED
        lj_alloc.c
        lj_api.c
        lj_asm.c
        lj_assert.c
        lj_bc.c
        lj_bcread.c
        lj_bcwrite.c
        lj_buf.c
        lj_carith.c
        lj_ccall.c
        lj_ccallback.c
        lj_cconv.c
        lj_cdata.c
        lj_char.c
        lj_clib.c
        lj_cparse.c
        lj_crecord.c
        lj_ctype.c
        lj_debug.c
        lj_dispatch.c
        lj_err.c
        lj_ffrecord.c
        lj_func.c
        lj_gc.c
        lj_gdbjit.c
        lj_ir.c
        lj_lex.c
        lj_lib.c
        lj_load.c
        lj_mcode.c
        lj_meta.c
        lj_obj.c
        lj_opt_dce.c
        lj_opt_fold.c
        lj_opt_loop.c
        lj_opt_mem.c
        lj_opt_narrow.c
        lj_opt_sink.c
        lj_opt_split.c
        lj_parse.c
        lj_prng.c
        lj_profile.c
        lj_record.c
        lj_serialize.c
        lj_snap.c
        lj_state.c
        lj_str.c
        lj_strfmt.c
        lj_strfmt_num.c
        lj_strscan.c
        lj_tab.c
        lj_trace.c
        lj_udata.c
        lj_vmevent.c
        lj_vmmath.c
        ${CMAKE_BINARY_DIR}/src/lj_vm.obj
        lib_aux.c
        lib_init.c
        ${ALL_LIB}
        version.rc
)
target_compile_definitions(lua51 PUBLIC LUA_BUILD_AS_DLL)
target_include_directories(
        lua51
        PUBLIC .
)
add_dependencies(lua51 buildvm)
add_custom_command(
        OUTPUT lj_vm.obj
        COMMAND ${BuildVM} -m peobj -o ${CMAKE_BINARY_DIR}/src/lj_vm.obj
        COMMENT Generate definition
        WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}"
        )

add_executable(luajit luajit.c version.rc)
target_link_libraries(luajit lua51)
add_dependencies(luajit lua51)

install(TARGETS luajit DESTINATION bin)
install(TARGETS lua51
        RUNTIME DESTINATION bin)
install(TARGETS lua51
        ARCHIVE DESTINATION lib)
install(FILES
        lauxlib.h
        lua.h
        lua.hpp
        luaconf.h
        luajit.h
        lualib.h
        DESTINATION include/luajit-2.1)
#install(FILES
#        ../etc/luajit.pc
#        DESTINATION lib/pkgconfig)
install(FILES
        jit/bc.lua
        jit/bcsave.lua
        jit/dis_arm.lua
        jit/dis_arm64.lua
        jit/dis_arm64be.lua
        jit/dis_mips.lua
        jit/dis_mips64.lua
        jit/dis_mips64el.lua
        jit/dis_mips64r6.lua
        jit/dis_mips64r6el.lua
        jit/dis_mipsel.lua
        jit/dis_ppc.lua
        jit/dis_x64.lua
        jit/dis_x86.lua
        jit/dump.lua
        jit/p.lua
        jit/v.lua
        jit/vmdef.lua
        jit/zone.lua
        DESTINATION share/luajit-2.1/jit)
install(FILES
        ../etc/luajit.1
        DESTINATION share/man/man1)
