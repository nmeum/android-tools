project (android-tools)
cmake_minimum_required(VERSION 3.1.0)

# Version of android-tools and the version of boringssl being used.
# See: https://android.googlesource.com/platform/external/boringssl/+/${ANDROID_VERSION}/BORINGSSL_REVISION
set(ANDROID_VERSION 8.1.0_r7)
set(BORINGSSL_VERSION 14308731e5446a73ac2258688a9688b524483cb6)

set(ANDROID_PATCH_DIR ${CMAKE_CURRENT_SOURCE_DIR}/patches)

add_subdirectory(vendor)

set(CPACK_SOURCE_GENERATOR "TXZ")
set(CPACK_SOURCE_PACKAGE_FILE_NAME "${PROJECT_NAME}-${ANDROID_VERSION}")
set(CPACK_SOURCE_IGNORE_FILES "/patches/" "/build/" "/.git/")
include(CPack)
