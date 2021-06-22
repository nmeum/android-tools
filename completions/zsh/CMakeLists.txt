set(ZSH_COMPLETION_DIR "${CMAKE_INSTALL_FULL_DATADIR}/zsh/site-functions")

configure_file(fastboot.in _fastboot)
# _adb completion is provided by zsh itself

install(FILES
	"${CMAKE_CURRENT_BINARY_DIR}/_fastboot"
	DESTINATION "${ZSH_COMPLETION_DIR}")
