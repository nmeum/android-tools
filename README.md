# android-tools

Git repository to make it easier to package certain command line
utilities provided by [android-tools][android-tools].

# Status

This is currently work in progress and more of an experiment. The
following tools can currently be build under Alpine Linux:

	* [x] adb
	* [x] fastboot
	* [ ] e2fsdroid
	* [ ] mkbootimg
	* [ ] mke2fs.android
	* [ ] ext2simg

# Motivation

[Many][void-linux] [Linux][arch-linux] [distribution][alpine-linux] have
a package called android-tools which ships essential android command
line tools like adb or fastboot. Sadly the upstream build system for
those tools is rather complex and doesn't allow building the command
line tools only.

Linux Distribution therefore mostly ship their own build systems for
building the command line utilities. This Repository aims to make
packaging of android command utilities easier by providing a simple
CMake based build system and a ready-to-use tarball which doesn't
require cloning all of the required git repositories manually.

# Dependencies

TODO

# Compilation

TODO

# Packaging

TODO

[android-tools]: https://sites.google.com/a/android.com/tools/
[void-linux]: https://github.com/voidlinux/void-packages/tree/master/srcpkgs/android-tools
[arch-linux]: https://www.archlinux.org/packages/community/x86_64/android-tools/
[alpine-linux]: https://pkgs.alpinelinux.org/package/edge/testing/x86_64/android-tools
