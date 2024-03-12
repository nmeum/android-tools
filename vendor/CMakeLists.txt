enable_language(C CXX)
include(CheckSymbolExists)

# Use C11 with GNU extensions.
set(CMAKE_C_STANDARD 11)
set(CMAKE_C_EXTENSIONS ON)
set(CMAKE_C_STANDARD_REQUIRED ON)

# Use C++20 with GNU extensions.
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_EXTENSIONS ON)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

if(APPLE)
	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -D_DARWIN_C_SOURCE -D__DARWIN_C_LEVEL=__DARWIN_C_FULL")
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -D_DARWIN_C_SOURCE -D__DARWIN_C_LEVEL=__DARWIN_C_FULL")
endif()

# Android seems to use various attributes supported by clang but not by
# GCC which causes it to emit lots of warnings. Since these attributes
# don't seem to effect runtime behaviour simply disable the warnings.
add_compile_options(-Wno-attributes)

# libfsmgr (required by fastboot) requires a 64-bit off_t for lseek. On
# 32-bit glibc platforms this is not the case by default.
add_compile_definitions(_FILE_OFFSET_BITS=64)

set(android-vendored
	avb
	adb
	base
	core
	extras
	libbase
	libufdt
	libziparchive
	logging
	mkbootimg
	native
	selinux
	f2fs-tools
	e2fsprogs
	incremental_delivery
	fmtlib
	boringssl)

# XXX: Consider using https://cmake.org/cmake/help/v3.0/module/ExternalProject.html

find_package(Git)
if(GIT_FOUND)
  list(APPEND PATCH_PROGRAM ${GIT_EXECUTABLE} am)
else()
  find_program(PATCH_EXECUTABLE patch REQUIRED)
  list(APPEND PATCH_PROGRAM ${PATCH_EXECUTABLE} -p1 -i)
endif()

if(ANDROID_TOOLS_PATCH_VENDOR AND EXISTS "${ANDROID_PATCH_DIR}/")
	if(GIT_FOUND)
	  execute_process(COMMAND git submodule --quiet update)
	endif()
	foreach(v ${android-vendored})
	  file(GLOB patches ${ANDROID_PATCH_DIR}/${v}/*.patch)
	  message(STATUS "Applying patches for: ${v}")
	  if(patches)
	  foreach(patch ${patches})
		execute_process(COMMAND ${PATCH_PROGRAM} ${patch}
		  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/${v}
		  RESULT_VARIABLE ret
		  COMMAND_ECHO STDOUT)
		if(NOT "${ret}" STREQUAL "0")
		  message(FATAL_ERROR "Couldn't apply patches for ${v}, ${ret}")
		endif()
	  endforeach()
	endif()
	endforeach(v)
endif()

add_subdirectory(boringssl EXCLUDE_FROM_ALL)
add_subdirectory(fmtlib EXCLUDE_FROM_ALL)

find_package(PkgConfig REQUIRED)
pkg_check_modules(libbrotlicommon REQUIRED IMPORTED_TARGET libbrotlicommon)
pkg_check_modules(libbrotlidec REQUIRED IMPORTED_TARGET libbrotlidec)
pkg_check_modules(libbrotlienc REQUIRED IMPORTED_TARGET libbrotlienc)
pkg_check_modules(liblz4 REQUIRED IMPORTED_TARGET liblz4)
pkg_check_modules(libpcre2-8 REQUIRED IMPORTED_TARGET libpcre2-8)
pkg_check_modules(libusb-1.0 REQUIRED IMPORTED_TARGET libusb-1.0)
pkg_check_modules(libzstd REQUIRED IMPORTED_TARGET libzstd)

find_package(Protobuf CONFIG)
find_package(Protobuf REQUIRED)
set(THREADS_PREFER_PTHREAD_FLAG ON)
find_package(Threads REQUIRED)

check_symbol_exists(strlcpy "string.h" HAVE_STRLCPY)
check_symbol_exists(reallocarray "stdlib.h" HAVE_REALLOCARRAY)

include_directories(${PROTOBUF_INCLUDE_DIRS})
include_directories(${CMAKE_CURRENT_BINARY_DIR})

include(CMakeLists.libbase.txt)
include(CMakeLists.libandroidfw.txt)
include(CMakeLists.adb.txt)
include(CMakeLists.sparse.txt)
include(CMakeLists.fastboot.txt)
include(CMakeLists.f2fstools.txt)
include(CMakeLists.mke2fs.txt)
include(CMakeLists.partition.txt)
include(CMakeLists.mkbootimg.txt)
include(CMakeLists.libufdt.txt)
include(CMakeLists.avb.txt)

# Targets which should be installed by `make install`.
install(TARGETS
	"${ANDROID_MKE2FS_NAME}"
	adb
	append2simg
	fastboot
	img2simg
	lpadd
	lpdump
	lpflash
	lpmake
	lpunpack
	make_f2fs
	sload_f2fs
	simg2img
	e2fsdroid
	ext2simg
	DESTINATION bin)

# Install common completion files.
install(FILES adb/adb.bash RENAME adb DESTINATION "${COMPLETION_COMMON_DIR}")
install(FILES core/fastboot/fastboot.bash RENAME fastboot DESTINATION "${COMPLETION_COMMON_DIR}")

# Install license files.
# Disabled for now, see https://github.com/nmeum/android-tools/issues/30#issuecomment-855365636
#set(LICENSE_DIR "${CMAKE_INSTALL_FULL_DATADIR}/licenses/android-tools")
#install(FILES core/NOTICE RENAME AOSP_LICENSE DESTINATION "${LICENSE_DIR}")
