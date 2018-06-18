# TODO: https://cmake.org/cmake/help/v3.0/module/ExternalProject.html
# To patch the submodule

add_subdirectory(boringssl)
set(android-vendored
	core
	extras
	selinux
	f2fs-tools
	e2fsprogs)

if(EXISTS "${ANDROID_PATCH_DIR}/")
	execute_process(COMMAND git -C ${CMAKE_CURRENT_SOURCE_DIR}/boringssl
		reset -q --hard ${BORINGSSL_VERSION})
	foreach(v ${android-vendored})
		execute_process(COMMAND git -C ${CMAKE_CURRENT_SOURCE_DIR}/${v}
			reset -q --hard android-${ANDROID_VERSION})

		file(GLOB patches ${ANDROID_PATCH_DIR}/${v}/*.patch)
		if(patches)
			message(STATUS "Applying patches for: ${v}")
			execute_process(COMMAND git -C
				${CMAKE_CURRENT_SOURCE_DIR}/${v} am ${patches})
		endif(patches)
	endforeach(v)
endif()

include(CMakeLists.adb.txt)
include(CMakeLists.fastboot.txt)
include(CMakeLists.mke2fs.txt)

# Targets which should be installed by `make install`.
install(TARGETS adb fastboot "${ANDROID_MKE2FS_NAME}" DESTINATION bin)
