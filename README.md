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

I have Android SDK and Android NDK installed.  NDK revision 9 or 10d.  Build
the standalone toolchain for Android NDK by running
`make-standalone-toolchain.sh` with the appropriate args as described
at: http://www.kandroid.org/ndk/docs/STANDALONE-TOOLCHAIN.html.  For example,

    $ ./android-ndk-r10d-linux-x86_64.bin
    $ cd android-ndk-r10d
    $ build/tools/make-standalone-toolchain.sh  --platform=android-19 --arch=x86 --toolchain=x86-4.9 --install-dir=/where/you/install/crosscompilers/i686-linux-android
    # (or for Arm you might choose: build/tools/make-standalone-toolchain.sh  --platform=android-19 --arch=arm --toolchain=arm-linux-androideabi-4.9 --install-dir=/where/you/install/crosscompilers/arm-linux-android)

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

The [`android-python-3.4.2.patch`](https://raw.githubusercontent.com/wandering-logic/android_x86_python-3.4/master/android-python-3.4.2.patch) included in this directory is a combination of the following:

* http://bugs.python.org/file37930/pw_gecos-field-workaround-with-config_ac-mods.patch
  from issue 20306.

*  http://bugs.python.org/file37042/time_select_audioop_ctypes_test_link_with_libm.patch
  from issue 21668.

* http://bugs.python.org/file27898/issue16353.diff from issue 16353.

* http://bugs.python.org/file37139/popen-no-hardcode-bin-sh.patch from
  issue 16255.

* A variety of patches to attack the issues with locales/langinfo and
  wide chars described in issues 22747 and 20305.

Additionally if you are trying to get Python-3.5 working then you'll
also need to apply:

* http://bugs.python.org/file36902/makefile-regen-fix.patch from
  Issue22625.  (This fixes a bug with cross compilation that was
  introduced after Python 3.4.2 released.)


So [download
`android-python-3.4.2.patch`](https://github.com/wandering-logic/android_x86_python-3.4/blob/master/android-python-3.4.2.patch) into the source directory
and apply it as follows:

* `patch -p1 < android-python-3.4.2.patch`
* `autoheader`
* `autoconf`

(The `autoheader` and `autoconf` commands rebuild the `pyconfig.h.in`
file and the `configure` script.  This is necessary because the patch
modifies `configure.ac`.  You get these by installing the `autoconf`
package.  `apt-get install autoconf` on Ubuntu/Debian, `yum install
autoconf` on CentOS/Fedora/Red Hat).

## CONFIGURE

In the directory that _contains_ the `Python-3.4.2` source tree do the
following:

1. `mkdir build-for-android`
2. `cd build-for-android`

3. `CPPFLAGS=-I/absolute/path/to/../Python-3.4.2/FIXLOCALE
   ../Python-3.4.2/configure --enable-shared
   --prefix=/absolute/path/to/install/directory
   --build=x86_64-linux-gnu --host=i686-linux-android --disable-ipv6
   ac_cv_file__dev_ptmx=no ac_cv_file__dev_ptc=no
   ac_cv_little_endian_double=yes --without-ensurepip`

(Explanation:

* `CPPFLAGS` sets the include path to some .h files needed by the
  patch.

* `../Python-3.4.2/configure` is the script you're running.

* `--enable-shared` tells `configure` you want to use shared libraries
  rather than static linking.

* `--prefix` is the path where the installation files will be placed
  on your _build_ machine.  If you have root access on the build
  machine then you probably want this to be something like
  `/data/python-3.4.2`.  (In which case you should `sudo mkdir /data`
  and `sudo chmod go+rwx /data`.)

* `--build` is the system you're building on (usually
  `x86_64-linux-gnu`).

* `--host` is the Android system you're targeting.  I've got
  `i686-linux-android`, but if you're targeting an Arm based
  phone/tablet then this needs to be something like
  `arm-linux-android`.

* `--disable-ipv6` is because I couldn't get the ipv6 support to compile.

* `ac_cv_file__dev_ptmx` and `ac_cv_file__dev_ptc` are to turn off
  some other feature that wouldn't properly cross compile.

* `ac_cv_little_endian_double=yes` is necessary on Android x86.  I
  don't know about Arm.  I'd start without it, and add it if any of
  the floating point unit tests are failing.

* `--without-ensure-pip` turns off some test that always fails if
  you're cross compiling.

## PATCH SOME MORE

4. download the patch [`after-config.patch`](https://raw.githubusercontent.com/wandering-logic/android_x86_python-3.4/master/after-config.patch) into the
   `build-for-android` directory.

5. In the `build-for-android` directory run `patch -p1 < after-config.patch`.

(This patch is a complete hack to deal with some more
cross-compilation bugs which I haven't had time to make real patches
for.)

## BUILD
6. `make && make install`

There may be some warnings, but there shouldn't be any errors.

## COPY TO ANDROID

How you've got your Android test box connected may be different than
mine.  I have the Android test box connected on my local intranet,
getting an IP address from the local DHCP server.

* `cd \data`
* `tar --bzip2 --create --file=py.tar.bz2 python-3.4.2/`
* `adb connect <ip.address.of.android.box>`
* `adb push py.tar.bz2 /data/`

## TEST

Now get a shell running on the Android machine.  You can do this from
a developer-enabled Android phone/tablet by running the `shell`
program.  I prefer to do it by running `adb shell` from my Linux
machine (where I have a much better keyboard and monitor.)

At the Android shell prompt:

* `PATH=/system/xbin/busybox:$PATH:/data/python-3.4.2/bin`
* `LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/data/python-3.4.2/lib`
* `cd /data/`
* `tar xf py.tar.bz2`

The `LD_LIBRARY_PATH` setting actually isn't required if you did your
build into `--prefix=/data/python-3.4.2`, but is required otherwise.

To see whether the most basic stuff is working type `python3` to bring
up the interactive Python prompt.  Once that is good then run `python3
-m test -x test_ctypes test_threading`.  (`test_ctypes` and
`test_threading` are failing catastrophically at the moment.)

This should take about 10 minutes and end with something like the
following:

```
327 tests OK.
21 tests failed:
    test_asyncio test_bytes test_capi test_concurrent_futures
    test_datetime test_distutils test_faulthandler test_fork1
    test_inspect test_io test_mmap test_multiprocessing_main_handling
    test_os test_posix test_posixpath test_pwd test_shutil test_socket
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

If you have any ideas about why the failing tests are failing we'd
love to hear from you!


