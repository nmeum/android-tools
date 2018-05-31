# TODO: https://cmake.org/cmake/help/v3.0/module/ExternalProject.html
# To patch the submodule

add_subdirectory(boringssl)
set(android-vendored
	core
	extras
	selinux
	f2fs-tools
	e2fsprogs)

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

add_definitions(-DADB_VERSION=${ANDROID_VERSION} -DADB_HOST=1
	-DFASTBOOT_VERSION=${ANDROID_VERSION})

add_library(libadb STATIC
	core/adb/console.cpp
	core/adb/socket_spec.cpp
	core/adb/commandline.cpp
	core/adb/adb_client.cpp
	core/adb/services.cpp
	core/adb/file_sync_client.cpp
	core/adb/sysdeps_unix.cpp
	core/adb/sysdeps/errno.cpp
	core/adb/client/main.cpp
	core/adb/client/usb_dispatch.cpp
	core/adb/client/usb_linux.cpp
	core/adb/client/usb_libusb.cpp
	core/adb/sysdeps/posix/network.cpp)

target_compile_definitions(libadb PRIVATE -D_GNU_SOURCE)
target_include_directories(libadb PUBLIC
	core/include core/base/include core/adb)

add_library(libbase STATIC
	core/base/file.cpp
	core/base/logging.cpp
	core/base/parsenetaddress.cpp
	core/base/stringprintf.cpp
	core/base/strings.cpp
	core/base/errors_unix.cpp
	core/base/test_utils.cpp)

target_include_directories(libbase PUBLIC
	core/base/include core/include)

add_library(liblog STATIC
	core/liblog/log_event_write.c
	core/liblog/fake_log_device.c
	core/liblog/log_event_list.c
	core/liblog/logger_write.c
	core/liblog/config_write.c
	core/liblog/config_read.c
	core/liblog/logger_lock.c
	core/liblog/local_logger.c
	core/liblog/fake_writer.c
	core/liblog/logger_name.c
	core/liblog/stderr_write.c
	core/liblog/logprint.c)

target_compile_definitions(liblog PRIVATE
	-DLIBLOG_LOG_TAG=1006 -D_XOPEN_SOURCE=700 -DFAKE_LOG_DEVICE=1)
target_include_directories(liblog PUBLIC
	core/log/include core/include)

add_library(libcutils STATIC
	core/libcutils/load_file.c
	core/libcutils/socket_local_client_unix.c
	core/libcutils/socket_network_client_unix.c
	core/libcutils/socket_local_server_unix.c
	core/libcutils/sockets_unix.cpp
	core/libcutils/socket_inaddr_any_server_unix.c
	core/libcutils/sockets.cpp
	core/libcutils/android_get_control_file.cpp
	core/libcutils/threads.c
	core/libcutils/fs_config.cpp
	core/libcutils/canned_fs_config.c)

target_compile_definitions(libcutils PRIVATE -D_GNU_SOURCE)
target_include_directories(libcutils PUBLIC
	core/libcutils/include core/include)

add_library(libdiagnoseusb STATIC
	core/adb/diagnose_usb.cpp)

target_include_directories(libdiagnoseusb PUBLIC
	core/include core/base/include)

add_library(libcrypto STATIC
	core/libcrypto_utils/android_pubkey.c)

target_include_directories(libcrypto PUBLIC
	core/libcrypto_utils/include boringssl/include)

add_executable(adb
	  core/adb/adb.cpp
	  core/adb/adb_io.cpp
	  core/adb/adb_listeners.cpp
	  core/adb/adb_trace.cpp
	  core/adb/adb_utils.cpp
	  core/adb/bugreport.cpp
	  core/adb/line_printer.cpp
	  core/adb/sockets.cpp
	  core/adb/transport.cpp
	  core/adb/transport_local.cpp
	  core/adb/transport_usb.cpp
	  core/adb/transport_mdns_unsupported.cpp
	  core/adb/fdevent.cpp
	  core/adb/adb_auth_host.cpp
	  core/adb/shell_service_protocol.cpp)

target_include_directories(adb PRIVATE
	core/include core/base/include core/adb
	core/libcrypto_utils/include boringssl/include)
target_link_libraries(adb libbase liblog libadb
	libdiagnoseusb libcrypto crypto pthread usb-1.0 libcutils)

add_library(libfastboot STATIC
	core/fastboot/protocol.cpp
	core/fastboot/engine.cpp
	core/fastboot/bootimg_utils.cpp
	core/fastboot/fastboot.cpp
	core/fastboot/util.cpp
	core/fastboot/fs.cpp
	core/fastboot/usb_linux.cpp
	core/fastboot/socket.cpp
	core/fastboot/tcp.cpp
	core/fastboot/udp.cpp)

target_include_directories(libfastboot PUBLIC
	core/base/include core/include core/adb core/libsparse/include
	core/mkbootimg extras/ext4_utils/include extras/f2fs_utils
	core/libziparchive/include)
target_compile_definitions(libfastboot PRIVATE
	-D_GNU_SOURCE -D_XOPEN_SOURCE=700 -DUSE_F2FS)

add_library(libsparse STATIC
	core/libsparse/backed_block.c
	core/libsparse/output_file.c
	core/libsparse/sparse.c
	core/libsparse/sparse_crc32.c
	core/libsparse/sparse_err.c
	core/libsparse/sparse_read.cpp)

target_include_directories(libsparse PUBLIC
	core/libsparse/include core/base/include)

add_library(libf2fs STATIC
	extras/f2fs_utils/f2fs_utils.c
	extras/f2fs_utils/f2fs_ioutils.c
	extras/f2fs_utils/f2fs_dlutils.c)

target_include_directories(libf2fs PUBLIC
	extras/f2fs_utils f2fs-tools/include f2fs-tools/mkfs
	core/libsparse/include selinux/libselinux/include)

add_library(libzip STATIC
	core/libziparchive/zip_archive.cc)

target_include_directories(libzip PUBLIC
	core/base/include core/include core/libziparchive/include)

add_library(libutil STATIC
	core/libutils/FileMap.cpp)

target_include_directories(libutil PUBLIC
	core/include)

add_library(libext4 STATIC
	extras/ext4_utils/make_ext4fs.c
	extras/ext4_utils/ext4fixup.c
	extras/ext4_utils/ext4_utils.c
	extras/ext4_utils/allocate.c
	extras/ext4_utils/contents.c
	extras/ext4_utils/extent.c
	extras/ext4_utils/indirect.c
	extras/ext4_utils/sha1.c
	extras/ext4_utils/wipe.c
	extras/ext4_utils/crc16.c
	extras/ext4_utils/ext4_sb.c)

target_include_directories(libext4 PUBLIC
	core/libsparse/include core/include selinux/libselinux/include extras/ext4_utils/include)

add_library(libselinux STATIC
	selinux/libselinux/src/callbacks.c
	selinux/libselinux/src/check_context.c
	selinux/libselinux/src/freecon.c
	selinux/libselinux/src/init.c
	selinux/libselinux/src/label.c
	selinux/libselinux/src/label_file.c
	selinux/libselinux/src/label_support.c
	selinux/libselinux/src/setrans_client.c
	selinux/libselinux/src/regex.c
	selinux/libselinux/src/matchpathcon.c
	selinux/libselinux/src/selinux_config.c
	selinux/libselinux/src/label_backends_android.c
	selinux/libselinux/src/canonicalize_context.c
	selinux/libselinux/src/lsetfilecon.c
	selinux/libselinux/src/policyvers.c
	selinux/libselinux/src/lgetfilecon.c
	selinux/libselinux/src/load_policy.c
	selinux/libselinux/src/seusers.c
	selinux/libselinux/src/sha1.c
	selinux/libselinux/src/booleans.c
	selinux/libselinux/src/disable.c
	selinux/libselinux/src/enabled.c
	selinux/libselinux/src/getenforce.c
	selinux/libselinux/src/setenforce.c)

target_compile_definitions(libselinux PRIVATE
	-DAUDITD_LOG_TAG=1003 -D_GNU_SOURCE -DHOST -DUSE_PCRE2
	-DNO_PERSISTENTLY_STORED_PATTERNS -DDISABLE_SETRANS
	-DDISABLE_BOOL -DNO_MEDIA_BACKEND -DNO_X_BACKEND -DNO_DB_BACKEND
	-DPCRE2_CODE_UNIT_WIDTH=8)
target_include_directories(libselinux PUBLIC
	selinux/libselinux/include selinux/libsepol/include)

add_library(libsepol
	selinux/libsepol/src/policydb_public.c
	selinux/libsepol/src/genbools.c
	selinux/libsepol/src/debug.c
	selinux/libsepol/src/policydb.c
	selinux/libsepol/src/conditional.c
	selinux/libsepol/src/services.c
	selinux/libsepol/src/ebitmap.c
	selinux/libsepol/src/util.c
	selinux/libsepol/src/assertion.c
	selinux/libsepol/src/avtab.c
	selinux/libsepol/src/hashtab.c
	selinux/libsepol/src/sidtab.c
	selinux/libsepol/src/context.c
	selinux/libsepol/src/genusers.c
	selinux/libsepol/src/context_record.c
	selinux/libsepol/src/mls.c
	selinux/libsepol/src/avrule_block.c
	selinux/libsepol/src/symtab.c
	selinux/libsepol/src/policydb_convert.c
	selinux/libsepol/src/write.c
	selinux/libsepol/src/constraint.c
	selinux/libsepol/src/expand.c
	selinux/libsepol/src/hierarchy.c)

target_include_directories(libsepol PUBLIC
	selinux/libsepol/include)

add_executable(fastboot
	core/fastboot/protocol.cpp
	core/fastboot/engine.cpp
	core/fastboot/bootimg_utils.cpp
	core/fastboot/fastboot.cpp
	core/fastboot/util.cpp
	core/fastboot/fs.cpp
	core/fastboot/usb_linux.cpp
	core/fastboot/socket.cpp
	core/fastboot/tcp.cpp
	core/fastboot/udp.cpp)

target_link_libraries(fastboot
	libsparse libzip libcutils liblog libutil libbase libext4
	libf2fs libselinux libsepol libfastboot libdiagnoseusb
	z pcre2-8 pthread dl)
