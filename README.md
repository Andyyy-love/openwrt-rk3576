## armv8.mk修改
```
路径  /work/rockchip/hejh/openwrt-rk3576/target/linux/rockchip/image/armv8.mk

define Device/radxa_rock-4d
  $(Device/rk3576)
  DEVICE_VENDOR := Radxa
  DEVICE_MODEL := ROCK 4D
endef
TARGET_DEVICES += radxa_rock-4d

define Device/igkboard_rk3576
  $(Device/rk3576)
  DEVICE_VENDOR := LingYun
  DEVICE_MODEL := IGKBoard RK3576
  DEVICE_DTS := igkboard-rk3576
  UBOOT_DEVICE_NAME := rock-4d-rk3576
  DEVICE_PACKAGES := kmod-r8169
endef
TARGET_DEVICES += igkboard_rk3576

define Device/radxa_rock-4se
  $(Device/rk3399)
  DEVICE_VENDOR := Radxa
  DEVICE_MODEL := ROCK 4SE
endef
TARGET_DEVICES += radxa_rock-4se
```
## uboot设备树位置
```
/work/rockchip/hejh/openwrt-rk3576/build_dir/target-aarch64_generic_musl/u-boot-rock-4d-rk3576/u-boot-2026.01/dts/upstream/src/arm64/rockchip/rk3576-rock-4d.dts
```
## uboot生成位置
```
rockchip@ubuntu22:/work/rockchip/hejh/openwrt-rk3576$ find staging_dir -name "rock-4d-rk3576-u-boot-rockchip.bin" -ls
 82748978   9216 -rw-r--r--   1 rockchip rockchip  9434112 5月  9 11:36 staging_dir/target-aarch64_generic_musl/image/rock-4d-rk3576-u-boot-rockchip.bin
rockchip@ubuntu22:/work/rockchip/hejh/openwrt-rk3576$ find staging_dir build_dir -name "*rock-4d-rk3576*u-boot-rockchip.bin" -ls
 82748978   9216 -rw-r--r--   1 rockchip rockchip  94
```
## uboot编译
### 查看uboot编译时间
```
ls -lh build_dir/target-aarch64_generic_musl/u-boot-rock-4d-rk3576/u-boot-2026.01/u-boot-rockchip.bin
ls -lh build_dir/target-aarch64_generic_musl/u-boot-rock-4d-rk3576/u-boot-2026.01/idbloader.img
ls -lh build_dir/target-aarch64_generic_musl/u-boot-rock-4d-rk3576/u-boot-2026.01/u-boot.itb
```
### 只重新编译uboot
```
make defconfig选择rock 4D编译uboot

make package/boot/uboot-rockchip/clean V=s
make package/boot/uboot-rockchip/compile V=s

检查生成时间
ls -lh build_dir/target-aarch64_generic_musl/u-boot-rock-4d-rk3576/u-boot-2026.01/u-boot-rockchip.bin
ls -lh build_dir/target-aarch64_generic_musl/u-boot-rock-4d-rk3576/u-boot-2026.01/idbloader.img
ls -lh build_dir/target-aarch64_generic_musl/u-boot-rock-4d-rk3576/u-boot-2026.01/u-boot.itb

同时检查最终staging文件
ls -lh staging_dir/target-aarch64_generic_musl/image/rock-4d-rk3576-u-boot-rockchip.bin

只编译 U-Boot 不一定会重新打包最终 .img，所以还要执行：
make -j$(nproc) V=s

然后看最终镜像时间：
ls -lh bin/targets/rockchip/armv8/
```
## 内核设备树位置
```
/work/rockchip/hejh/openwrt-rk3576/build_dir/target-aarch64_generic_musl/linux-rockchip_armv8/linux-6.12.85/arch/arm64/boot/dts/rockchip/rk3576-rock-4d.dts
```

## 重新生成 rootfs 和镜像
```
rm -f staging_dir/target-aarch64_generic_musl/stamp/.*target*

make -j$(nproc) V=s
这个会重新生成：
build_dir/target-aarch64_generic_musl/root-rockchip/

eg：
rockchip@ubuntu22:/work/rockchip/hejh/openwrt-rk3576$ ls build_dir/target-aarch64_generic_musl/root-rockchip/
bin  dev  etc  lib  lib64  mnt  overlay  proc  rom  root  sbin  sys  tmp  usr  var  www
rockchip@ubuntu22:/work/rockchip/hejh/openwrt-rk3576$ ls build_dir/target-aarch64_generic_musl/root-rockchip/lib/firmware/brcm/
brcmfmac43143.bin  brcmfmac43236b.bin  brcmfmac43456-sdio.bin  brcmfmac43456-sdio.clm_blob  brcmfmac43456-sdio.txt
rockchip@ubuntu22:/work/rockchip/hejh/openwrt-rk3576$ 

以及最终镜像：
bin/targets/rockchip/armv8/*.img
bin/targets/rockchip/armv8/*.img.gz

```
## 一些bug
U-Boot 时间不变是正常的，不是旧 U-Boot 没更新。
```
rockchip@ubuntu22:/work/rockchip/hejh/openwrt-rk3576$ date -u -d @$(./scripts/get_source_date_epoch.sh)
2026年 05月 08日 星期五 09:31:34 UTC
rockchip@ubuntu22:/work/rockchip/hejh/openwrt-rk3576$ strings staging_dir/target-aarch64_generic_musl/image/rock-4d-rk3576-u-boot-rockchip.bin | grep "U-Boot 2026"
U-Boot 2026.01-OpenWrt-r34345-8b393f99fd (May 08 2026 - 09:31:34 +0000)
rockchip@ubuntu22:/work/rockchip/hejh/openwrt-rk3576$

```
## 测试内核设备树补丁能不能自动打进去
```
先清内核：

make target/linux/clean V=s

然后只准备内核源码和补丁：

make target/linux/prepare V=s

如果没有报：

Patch failed

说明补丁能正常打进去。

然后确认文件重新出现：

ls build_dir/target-aarch64_generic_musl/linux-rockchip_armv8/linux-6.12.85/arch/arm64/boot/dts/rockchip/rk3576-rk806.dtsi

ls build_dir/target-aarch64_generic_musl/linux-rockchip_armv8/linux-6.12.85/arch/arm64/boot/dts/rockchip/igkboard-rk3576.dts
```

![OpenWrt logo](include/logo.png)

OpenWrt Project is a Linux operating system targeting embedded devices. Instead
of trying to create a single, static firmware, OpenWrt provides a fully
writable filesystem with package management. This frees you from the
application selection and configuration provided by the vendor and allows you
to customize the device through the use of packages to suit any application.
For developers, OpenWrt is the framework to build an application without having
to build a complete firmware around it; for users this means the ability for
full customization, to use the device in ways never envisioned.

Sunshine!

## Download

Built firmware images are available for many architectures and come with a
package selection to be used as WiFi home router. To quickly find a factory
image usable to migrate from a vendor stock firmware to OpenWrt, try the
*Firmware Selector*.

* [OpenWrt Firmware Selector](https://firmware-selector.openwrt.org/)

If your device is supported, please follow the **Info** link to see install
instructions or consult the support resources listed below.

## 

An advanced user may require additional or specific package. (Toolchain, SDK, ...) For everything else than simple firmware download, try the wiki download page:

* [OpenWrt Wiki Download](https://openwrt.org/downloads)

## Development

To build your own firmware you need a GNU/Linux, BSD or macOS system (case
sensitive filesystem required). Cygwin is unsupported because of the lack of a
case sensitive file system.

### Requirements

You need the following tools to compile OpenWrt, the package names vary between
distributions. A complete list with distribution specific packages is found in
the [Build System Setup](https://openwrt.org/docs/guide-developer/build-system/install-buildsystem)
documentation.

```
binutils bzip2 diff find flex gawk gcc-6+ getopt grep install libc-dev libz-dev
make4.1+ perl python3.7+ rsync subversion unzip which
```

### Quickstart

1. Run `./scripts/feeds update -a` to obtain all the latest package definitions
   defined in feeds.conf / feeds.conf.default

2. Run `./scripts/feeds install -a` to install symlinks for all obtained
   packages into package/feeds/

3. Run `make menuconfig` to select your preferred configuration for the
   toolchain, target system & firmware packages.

4. Run `make` to build your firmware. This will download all sources, build the
   cross-compile toolchain and then cross-compile the GNU/Linux kernel & all chosen
   applications for your target system.

### Related Repositories

The main repository uses multiple sub-repositories to manage packages of
different categories. All packages are installed via the OpenWrt package
manager called `opkg`. If you're looking to develop the web interface or port
packages to OpenWrt, please find the fitting repository below.

* [LuCI Web Interface](https://github.com/openwrt/luci): Modern and modular
  interface to control the device via a web browser.

* [OpenWrt Packages](https://github.com/openwrt/packages): Community repository
  of ported packages.

* [OpenWrt Routing](https://github.com/openwrt/routing): Packages specifically
  focused on (mesh) routing.

* [OpenWrt Video](https://github.com/openwrt/video): Packages specifically
  focused on display servers and clients (Xorg and Wayland).

## Support Information

For a list of supported devices see the [OpenWrt Hardware Database](https://openwrt.org/supported_devices)

### Documentation

* [Quick Start Guide](https://openwrt.org/docs/guide-quick-start/start)
* [User Guide](https://openwrt.org/docs/guide-user/start)
* [Developer Documentation](https://openwrt.org/docs/guide-developer/start)
* [Technical Reference](https://openwrt.org/docs/techref/start)

### Support Community

* [Forum](https://forum.openwrt.org): For usage, projects, discussions and hardware advise.
* [Support Chat](https://webchat.oftc.net/#openwrt): Channel `#openwrt` on **oftc.net**.

### Developer Community

* [Bug Reports](https://bugs.openwrt.org): Report bugs in OpenWrt
* [Dev Mailing List](https://lists.openwrt.org/mailman/listinfo/openwrt-devel): Send patches
* [Dev Chat](https://webchat.oftc.net/#openwrt-devel): Channel `#openwrt-devel` on **oftc.net**.

## License

OpenWrt is licensed under GPL-2.0
