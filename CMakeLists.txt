cmake_minimum_required(VERSION 2.8)

PROJECT(dsn.zookeeper)

set(src 
"src/hashtable/hashtable_itr.c"
"src/hashtable/hashtable.c"
"src/mt_adaptor.c"
"src/recordio.c"
"src/zk_hashtable.c"
"src/zk_log.c"
"src/zookeeper.c"
)

set(src_generated "generated/zookeeper.jute.c")

ADD_DEFINITIONS(-DTHREADED)
if (NOT WIN32)
	ADD_DEFINITIONS(-fPIC)
endif()

INCLUDE_DIRECTORIES("include")

macro(ms_link_static_runtime FLAG_VAR)
    if(MSVC)
        if(${FLAG_VAR} MATCHES "/MD")
            string(REPLACE "/MD"  "/MT" "${FLAG_VAR}" "${${FLAG_VAR}}")
            #Save persistently
            set(${FLAG_VAR} ${${FLAG_VAR}} CACHE STRING "" FORCE)
        endif()
    endif()
endmacro(ms_link_static_runtime)

macro(ms_replace_compiler_flags REPLACE_OPTION)
    set(SUFFIXES "")
    if((NOT DEFINED CMAKE_CONFIGURATION_TYPES) OR (CMAKE_CONFIGURATION_TYPES STREQUAL ""))
        #set(SUFFIXES "_DEBUG" "_RELEASE" "_MINSIZEREL" "_RELWITHDEBINFO")
        if((DEFINED CMAKE_BUILD_TYPE) AND (NOT (CMAKE_BUILD_TYPE STREQUAL "")))
            string(TOUPPER ${CMAKE_BUILD_TYPE} SUFFIXES)
            set(SUFFIXES "_${SUFFIXES}")
        endif()
    else()
        foreach(SUFFIX ${CMAKE_CONFIGURATION_TYPES})
            string(TOUPPER ${SUFFIX} SUFFIX)
            set(SUFFIXES ${SUFFIXES} "_${SUFFIX}")
        endforeach()
    endif()

    foreach(SUFFIX "" ${SUFFIXES})
        foreach(LANG C CXX)
            set(FLAG_VAR "CMAKE_${LANG}_FLAGS${SUFFIX}")
            if(${REPLACE_OPTION} STREQUAL "STATIC_LINK")
                ms_link_static_runtime(${FLAG_VAR})
            endif()
        endforeach()
        #message(STATUS ${FLAG_VAR} ":" ${${FLAG_VAR}})
    endforeach()
endmacro(ms_replace_compiler_flags)

if(MSVC)
	add_definitions(-D_CRT_SECURE_NO_WARNINGS)
	add_definitions(-DWIN32_LEAN_AND_MEAN)        
	add_definitions(-D_CRT_NONSTDC_NO_DEPRECATE)
	add_definitions(-D_WINSOCK_DEPRECATED_NO_WARNINGS=1)
	add_definitions(-D_WIN32_WINNT=0x0600)
	add_definitions(-D_UNICODE)
	add_definitions(-DUNICODE)
	add_definitions(-DDLL_EXPORT)
	add_compile_options(-MP)
	ms_replace_compiler_flags("STATIC_LINK")
	set(src ${src} "src/winport.c")
	ADD_LIBRARY(zookeeper_mt SHARED ${src} ${src_generated})
	target_link_libraries(zookeeper_mt PRIVATE "ws2_32.lib")	
else()
	ADD_LIBRARY(zookeeper_mt STATIC ${src} ${src_generated})
endif()
		

