# Python 3.4 for Android x86

Patches to get cross compilation and unit tests running for Python 3.4
for Android x86.  (Should also work for Android ARM, but I haven't
tested.)

Here's how to get Python 3.4.2 cross-compiled and running on Android.
I am targetting Android 4.4.2 (KitKat) running 32-bit on an Intel Core
i5-4250U.

## SETUP

My build machine is Linux (Ubuntu 14.04.1 LTS on an x86_64 machine).
Any Linux with glibc 2.7 or later should work (Debian 5+, Ubuntu
8.04+, SLES 11+, Fedora 8+, RHEL 6+, Centos 6+), but I haven’t tested
with anything other than Ubuntu 14.04.

I have Android SDK and Android NDK installed.  NDK revision 9+.  Build
the standalone toolchain for Android NDK by running
`make-standalone-toolchain.sh` with the appropriate args as described
at: http://www.kandroid.org/ndk/docs/STANDALONE-TOOLCHAIN.html.

The `bin/` subdir of the standalone toolchain should be on your path.
The Android sdk `adb` tool should be on your path.  (This is usually
found in the `sdk/platform-tools/` subdirectory of the SDK).

## DOWNLOAD

Download https://www.python.org/ftp/python/3.4.2/Python-3.4.2.tar.xz
and untar it.

You need a recent version of Python 3 running on the Build machine.
The easiest way to do this is by compiling and installing a local
version of the same Python you are trying to compile for Android.

1.  `mkdir build-build`
2.  `cd build-build`
3.  `../Python-3.4.2/configure  --enable-shared --prefix=</absolute/path/to/writable/install-dir>`
4.  `make`
5.  `make install`
6.  Add `</absolute/path/to/writable/install-dir>/bin` to the front of your `PATH`
7.  Add `</absolute/path/to/writable/install-dir>/lib` to the front of your `LD_LIBRARY_PATH`

## PATCH

The patch included in this directory is a combination of the following:

* http://bugs.python.org/file37930/pw_gecos-field-workaround-with-config_ac-mods.patch from issue 20306.
* http://bugs.python.org/file37042/time_select_audioop_ctypes_test_link_with_libm.patch from issue 21668.
* http://bugs.python.org/file27898/issue16353.diff from issue 16353.
* http://bugs.python.org/file37139/popen-no-hardcode-bin-sh.patch from issue 16255.
* A variety of patches to attack the issues with locales/langinfo and wide chars described in issues 22747 and 20305.

Additionally if you are trying to get Python-3.5 working then you'll
also need to apply:

* http://bugs.python.org/file36902/makefile-regen-fix.patch from Issue22625.


So download the patch and apply it as follows:

* `cd Python-3.4.2`
* `patch -p1 < <path/to/the/patch>`

## CONFIGURE

1. `mkdir build-for-android`
2. `cd build-for-android`
3.  `CPPFLAGS="-I/absolute/path/to/../Python-3.4.2/FIXLOCALE -DENABLE_ANDROID_HACKS" ../Python-3.4.2/configure --enable-shared --prefix=/absolute/path/to/install/directory --build=x86_64-linux-gnu --host=i686-linux-android --disable-ipv6 ac_cv_file__dev_ptmx=no ac_cv_file__dev_ptc=no ac_cv_little_endian_double=yes --without-ensurepip`

## PATCH SOME MORE
4. download the patch `after-config.patch`.
5. In the `build-for-android` directory run `patch -p1 < after-config.patch`.

## BUILD
6. make && make install

## TEST

Run `python3 -m test -x test_ctypes test_threading`

This should take about 10 minutes and end with something like the
following:

```
328 tests OK.
20 tests failed:
    test_asyncio test_bytes test_capi test_concurrent_futures
    test_datetime test_distutils test_faulthandler test_inspect
    test_io test_mmap test_multiprocessing_main_handling test_os
    test_posix test_posixpath test_pwd test_shutil test_socket
    test_subprocess test_sys test_unicode
38 tests skipped:
    test_bz2 test_crypt test_curses test_dbm_gnu test_dbm_ndbm
    test_devpoll test_gdb test_grp test_idle test_ioctl test_kqueue
    test_lzma test_msilib test_nis test_openpty test_ossaudiodev
    test_pep277 test_pty test_readline test_smtpnet test_socketserver
    test_spwd test_sqlite test_ssl test_startfile test_tcl
    test_timeout test_tk test_ttk_guionly test_ttk_textonly
    test_unicode_file test_urllib2net test_urllibnet test_wait4
    test_winreg test_winsound test_xmlrpc_net test_zipfile64
```