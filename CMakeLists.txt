cmake_minimum_required(VERSION 3.14 FATAL_ERROR)

project(ICMake VERSION 1.0.0.0)

set(test_1 12 CACHE STRING test_1 FORCE)

set(specialStr "aaa;bbb")

message(STATUS "${specialStr}")

function(PrintVar var)
    message(STATUS "${var}: ${${var}}")
endfunction()

function(PrintValue value)
    message(STATUS "${value}")
endfunction()

set(num 2)
PrintVar(num)

printValue(${num})

set(w_list 12 12 12)

printValue(w_list)

set(i 0)
while( i LESS 2)
    # ...
    math(EXPR i "${i} + 1")
    MESSAGE(STATUS ${i})
endwhile()


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

MESSAGE(STATUS "rdir:${${rdir}}")

#target_sources(test PUBLIC include/test.h)
#add_library(test)

add_executable(main main.cpp)
