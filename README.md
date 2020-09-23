# Android kernel (v4.4.146) for the Volla Phone

## Branches
* [android-9.0](../../tree/android-9.0) — Android 9 kernel source with many build fixes and some cleanup
* [halium-9.0](../../tree/halium-9.0) — Same as android-9.0, but with Halium 9 patches for e.g. Ubuntu Touch

## Building

### Environment
The currently used & tested working build env consists of:
* An x86-64 Ubuntu 18.04 (-based) distro
* binutils 2.30
* GCC 7.5.0

### Dependencies
```bash
sudo apt install -y build-essential gcc-aarch64-linux-gnu kmod libssl-dev bc python curl mkbootimg
```

### Kernel build
```bash
alias make="/usr/bin/make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- O=out"
alias mka="make -j$(nproc)"

make k63v2_64_bsp_defconfig
mka
```

### Halium ramdisk
```bash
curl -L https://github.com/Halium/initramfs-tools-halium/releases/download/continuous/initrd.img-touch-arm64 -o halium-ramdisk.tar.gz
```
**NOTE:** Replace `/dev/null` below with `halium-ramdisk.tar.gz` (make sure your branch is `halium-9.0`!)

### Android bootimg
```bash
mkbootimg --kernel 'out/arch/arm64/boot/Image.gz-dtb' \
          --ramdisk '/dev/null' \
          --cmdline 'bootopt=64S3,32N2,64N2 systempart=/dev/disk/by-partlabel/system' \
          --base '0x40078000' \
          --kernel_offset '0x8000' \
          --ramdisk_offset '0x14f88000' \
          --tags_offset '0x13f88000' \
          --pagesize '2048' \
          -o boot.img
```

### Packaging kernel modules
```bash
make INSTALL_MOD_PATH=rootfs modules_install

MODULES=out/rootfs/lib/modules
KVER=$(ls $MODULES | head -1)
FILES=$(find $MODULES/$KVER/ -type f -name *.ko)
FILES=$(ls $FILES $MODULES/$KVER/{modules.alias,modules.dep})

mkdir /tmp/kernel_modules
for f in $FILES; do
  cp $f /tmp/kernel_modules/
done
tar -C /tmp/kernel_modules -czvf kernel-modules.tar.gz .
rm -r /tmp/kernel_modules
```
**NOTE:** The created `kernel-modules.tar.gz` should be extracted to `/system/lib/modules` replacing modules for the default kernel!

## Links
* [Default README](README)
* [HelloVolla's kernel tree](https://github.com/HelloVolla/android_kernel_volla_mt6763)
* [Upstream linux-stable 4.4.y tree](https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/log/?h=linux-4.4.y)
* [Ubuntu 18.04 Minimal Cloud Image](https://partner-images.canonical.com/core/bionic/current/ubuntu-bionic-core-cloudimg-amd64-root.tar.gz)
