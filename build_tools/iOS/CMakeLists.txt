cmake_minimum_required(VERSION 3.6)

project(external)
include(config.cmake)
set(IPHONEOS_DEPLOYMENT_TARGET $ENV{DEPLOYMENT_TARGET})
set(CMAKE_XCODE_ATTRIBUTE_IPHONEOS_DEPLOYMENT_TARGET $ENV{DEPLOYMENT_TARGET})

set(LINK_OPTS -all_load)
if ("${IOS_PLATFORM}" STREQUAL "SIMULATOR64")
    set(LINK_OPTS "${LINK_OPTS},-read_only_relocs,suppress")
    #else()
    #    set(LINK_OPTS "${LINK_OPTS},-fembed-bitcode")
endif ()

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fembed-bitcode")
set(LIBRARYS ${SRC_LIBRARIES})

link_directories(${SRC_LIBRARIES_DIR})
add_library(${LIB_NAME} SHARED
        dummy.c
        )
if (IOS)
set_target_properties(${LIB_NAME} PROPERTIES
        FRAMEWORK TRUE
        FRAMEWORK_VERSION A
        MACOSX_FRAMEWORK_IDENTIFIER com.alibaba.AliyunPlayer.${LIB_NAME}
        LINK_FLAGS "-Wl,${LINK_OPTS}"
        LINKER_LANGUAGE C
        MACOSX_FRAMEWORK_BUNDLE_VERSION 1
        MACOSX_FRAMEWORK_SHORT_VERSION_STRING 1.0
        INSTALL_NAME_DIR @rpath
        #        MACOSX_FRAMEWORK_INFO_PLIST Info.plist
        #        PUBLIC_HEADER dynamicFramework.h
        # XCODE_ATTRIBUTE_CODE_SIGN_IDENTITY "iPhone Developer"
        XCODE_ATTRIBUTE_GCC_GENERATE_DEBUGGING_SYMBOLS[variant=MinSizeRel] YES
        XCODE_ATTRIBUTE_ENABLE_BITCODE "YES"
        XCODE_ATTRIBUTE_BITCODE_GENERATION_MODE[variant=MinSizeRel] bitcode
        )
else ()
    set_target_properties(${LIB_NAME} PROPERTIES
            FRAMEWORK TRUE
            FRAMEWORK_VERSION A
            MACOSX_FRAMEWORK_IDENTIFIER com.alibaba.AliyunPlayer.${LIB_NAME}
            LINK_FLAGS "-Wl,${LINK_OPTS}"
            LINKER_LANGUAGE C
            MACOSX_FRAMEWORK_BUNDLE_VERSION 1
            MACOSX_FRAMEWORK_SHORT_VERSION_STRING 1.0
            INSTALL_NAME_DIR @rpath
            )
endif ()
#if (IOS)
#    macro(set_xcode_property TARGET XCODE_PROPERTY XCODE_VALUE)
#        set_property(TARGET ${TARGET} PROPERTY XCODE_ATTRIBUTE_${XCODE_PROPERTY}
#                ${XCODE_VALUE})
#    endmacro(set_xcode_property)
#endif ()
#
#set_xcode_property(${LIB_NAME} GCC_GENERATE_DEBUGGING_SYMBOLS[variant=Release] YES)
#if (IOS)
#    set_xcode_property(${LIB_NAME} ENABLE_BITCODE "YES")
#    set_xcode_property(${LIB_NAME} BITCODE_GENERATION_MODE bitcode)
#endif ()
target_link_libraries(${LIB_NAME} PRIVATE ${LIBRARYS})

add_custom_command(
        TARGET ${LIB_NAME}
        POST_BUILD
        COMMAND install_name_tool -id \"@rpath/${LIB_NAME}.framework/${LIB_NAME}\"
        $<TARGET_FILE:${LIB_NAME}>
)
