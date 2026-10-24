# BusyBox HTTP/CGI support

Enable the HTTP applet and native CGI execution in the tracked
`sysdrv/tools/board/buildroot/busybox.config`:

```text
CONFIG_HTTPD=y
CONFIG_FEATURE_HTTPD_PORT_DEFAULT=80
CONFIG_FEATURE_HTTPD_CGI=y
```

This shared BusyBox config is copied by both fresh-extraction paths in
`sysdrv/Makefile`: normal `buildroot` creation and `buildroot_create`.
The change therefore applies to both SDK Buildroot defconfigs that use
this file, not only the locally selected `luckfox_pico_w_defconfig`.
The vendor archive is unchanged. Existing extracted Buildroot trees are
not automatically refreshed by those fresh-extraction copy operations.

For an existing, configured extraction of the selected bundled version,
update its configuration and rebuild only BusyBox from the SDK root:

```sh
cp sysdrv/tools/board/buildroot/busybox.config \
  sysdrv/source/buildroot/buildroot-2023.02.6/package/busybox/busybox.config
make -C sysdrv/source/buildroot/buildroot-2023.02.6 busybox-reconfigure
```

This enables a capability; it does not launch a server or choose a bind
address. The HTTP applet is installed as `/usr/sbin/httpd`, linked to
`/bin/busybox`. Applications provide their document root, CGI executable,
configuration and startup policy separately. The default HTTP port is 80;
a server can explicitly choose another port. No TLS or Basic Authentication
options are enabled by this change. Compiler flags, global LTO settings,
board/device-tree selection and vendor ipcweb remain unchanged.

## Validation (2026-10-24)

The targeted `busybox-reconfigure` command passed (exit 0) on the existing
native Lubuntu build using SDK ARM GCC 8.3 and BusyBox 1.36.1. The generated
BusyBox config contains all three settings, and the Buildroot target rootfs
contains `/usr/sbin/httpd -> ../../bin/busybox`. Equivalently stripped ARM
binary copies measured 857296 bytes before and 865616 bytes after (+8320).
The existing host compatibility fixes remain prerequisites for this host.
No automated tests, Python tooling, target execution, image repackaging,
flashing, network listener or hardware operation was performed.
