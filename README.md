# android_x86_python-3.4

Patches to get cross compilation and unit tests running for Python 3.4
for Android x86.  (Should also work for Android ARM, but I haven't
tested.)

Here's how to get Python 3.4.2 cross-compiled and running on Android.

SETUP:

My build machine is Linux (Ubuntu 14.04.1 LTS on an x86_64 machine).
Any Linux with glibc 2.7 or later should work (Debian 5+, Ubuntu
8.04+, SLES 11+, Fedora 8+, RHEL 6+, Centos 6+), but I haven’t tested
with anything other than Ubuntu 14.04.

I have Android SDK and Android NDK installed.  NDK revision 9+.  Build
the standalone toolchain for Android NDK by running the
“make-standalone-toolchain.sh” with the appropriate args as described
at: http://www.kandroid.org/ndk/docs/STANDALONE-TOOLCHAIN.html.

The bin/ subdir of the standalone toolchain should be on your path adb
should be on your path.  (This is usually found in the
sdk/platform-tools/ subdirectory of the SDK).

Download https://www.python.org/ftp/python/3.4.2/Python-3.4.2.tar.xz
and untar it.

You need a recent version of Python 3 running on the Build machine.
The easiest way to do this is by compiling and installing a local
version of the same Python you are trying to compile for Android.

1.  mkdir build-build
2.  cd build-build
3.  ../Python-3.4.2/configure  --enable-shared --prefix=</absolute/path/to/writable/install-dir>
4.  make
5.  make install
6.  Add </absolute/path/to/writable/install-dir>/bin to the front of your PATH
7.  Add </absolute/path/to/writable/install-dir>/lib to the front of your LD_LIBRARY_PATH

PATCHES:

1. Download http://bugs.python.org/file37930/pw_gecos-field-workaround-with-config_ac-mods.patch from issue 20306.
2. Apply it: (in the Python-3.4.2/ source directory run “patch –p1 < <path to .patch file>”)
3. Download http://bugs.python.org/file37042/time_select_audioop_ctypes_test_link_with_libm.patch from issue 21668 and apply it.
4. Download http://bugs.python.org/file27898/issue16353.diff from issue 16353 and apply it (one of the hunks in Lib/test/test_os.py may fail, don’t worry about it too much for now.)
5. Download http://bugs.python.org/file37139/popen-no-hardcode-bin-sh.patch from issue 16255 and apply it.
6. If you are trying to get Python-3.5 working then you’ll also need to apply http://bugs.python.org/file36902/makefile-regen-fix.patch from Issue22625.
7. Do something about the following functions that are not included in Android’s Bionic libc: 


