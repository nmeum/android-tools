set(ZSH_COMPLETION_DIR "${CMAKE_INSTALL_FULL_DATADIR}/zsh/site-functions")

configure_file(adb.in _adb)
configure_file(fastboot.in _fastboot)

install(FILES
	"${CMAKE_CURRENT_BINARY_DIR}/_adb"
	"${CMAKE_CURRENT_BINARY_DIR}/_fastboot"
	DESTINATION "${ZSH_COMPLETION_DIR}")
