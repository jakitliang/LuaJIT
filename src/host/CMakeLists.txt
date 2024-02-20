
add_executable(minilua minilua.c)

set(SRC_DIR ${CMAKE_CURRENT_SOURCE_DIR}/../src)

string(TIMESTAMP REL_VER "%s")

add_custom_command(TARGET minilua POST_BUILD
        COMMAND echo ${REL_VER} > luajit_relver.txt
        COMMAND ${MiniLua} ${DASM} -LN ${DASMFLAGS} -o host/buildvm_arch.h ${DASC}
        COMMAND ${MiniLua} host/genversion.lua
        COMMENT "Generate version"
        WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/.."
        )

list(
        APPEND minilua-list
        ${CMAKE_CURRENT_SOURCE_DIR}/../luajit_relver.txt
        ${CMAKE_CURRENT_SOURCE_DIR}/../luajit.h
        ${CMAKE_CURRENT_SOURCE_DIR}/buildvm_arch.h
)

set_property(
        TARGET minilua
        APPEND
        PROPERTY ADDITIONAL_CLEAN_FILES ${minilua-list}
)

add_executable(buildvm buildvm.c buildvm_asm.c buildvm_fold.c buildvm_lib.c buildvm_peobj.c)
add_dependencies(buildvm minilua)
target_include_directories(buildvm PRIVATE .)
target_include_directories(buildvm PRIVATE ..)
add_custom_command(TARGET buildvm POST_BUILD
#        COMMAND ${BuildVM} -m peobj -o lj_vm.obj
        COMMAND ${BuildVM} -m bcdef -o lj_bcdef.h ${ALL_LIB}
        COMMAND ${BuildVM} -m ffdef -o lj_ffdef.h ${ALL_LIB}
        COMMAND ${BuildVM} -m libdef -o lj_libdef.h ${ALL_LIB}
        COMMAND ${BuildVM} -m recdef -o lj_recdef.h ${ALL_LIB}
        COMMAND ${BuildVM} -m vmdef -o jit/vmdef.lua ${ALL_LIB}
        COMMAND ${BuildVM} -m folddef -o lj_folddef.h lj_opt_fold.c
        COMMENT Generate definition
        WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/.."
        )

list(
        APPEND buildvm-list
#        ${CMAKE_CURRENT_SOURCE_DIR}/../lj_vm.obj
        ${CMAKE_CURRENT_SOURCE_DIR}/../lj_bcdef.h
        ${CMAKE_CURRENT_SOURCE_DIR}/../lj_ffdef.h
        ${CMAKE_CURRENT_SOURCE_DIR}/../lj_libdef.h
        ${CMAKE_CURRENT_SOURCE_DIR}/../lj_recdef.h
        ${CMAKE_CURRENT_SOURCE_DIR}/../jit/vmdef.lua
        ${CMAKE_CURRENT_SOURCE_DIR}/../lj_folddef.h
)
set_property(
        TARGET buildvm
        APPEND
        PROPERTY ADDITIONAL_CLEAN_FILES ${buildvm-list})
