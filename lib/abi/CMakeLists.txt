
# Detect if we are building in the same configuration used to generate
# the abilist files.
if (DEFINED TARGET_TRIPLE
    AND EXISTS "${CMAKE_CURRENT_LIST_DIR}/${TARGET_TRIPLE}.abilist"
    AND TARGET cxx_shared
    AND ("${LIBCXX_CXX_ABI_LIBNAME}" STREQUAL "libcxxabi" OR
         (APPLE AND "${LIBCXX_CXX_ABI_LIBNAME}" STREQUAL "default"))
    AND NOT LIBCXX_ABI_UNSTABLE
    AND LIBCXX_ABI_VERSION EQUAL "1")
    set(LIBCXX_HAS_ABILIST_CONFIGURATION 1 CACHE INTERNAL "")
else()
    if (NOT DEFINED LIBCXX_HAS_ABILIST_CONFIGURATION)
        message(STATUS "libc++ configuration differs from the abilist configuration. "
                "check-cxx-abilist target is not supported")
    endif()
    set(LIBCXX_HAS_ABILIST_CONFIGURATION 0 CACHE INTERNAL "")
endif()


if (LIBCXX_HAS_ABILIST_CONFIGURATION)
    set(ABILIST_FILE "${CMAKE_CURRENT_LIST_DIR}/${TARGET_TRIPLE}.abilist")
    set(SYMDIFF_EXE "${LIBCXX_SOURCE_DIR}/utils/sym_check/sym_diff.py")
    add_custom_target(check-cxx-abilist
            ${SYMDIFF_EXE} --only-stdlib-symbols ${ABILIST_FILE} $<TARGET_SONAME_FILE:cxx_shared>
            DEPENDS cxx_shared
            COMMENT "Testing ABI compatibility...")
endif()
