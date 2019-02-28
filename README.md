# Building the mainline SELinux kernel
### Set-up and configure source
Download the kernel source [selinux-4.20-rc7.tar.gz](https://git.kernel.org/pub/scm/linux/kernel/git/pcmoore/selinux.git/snapshot/selinux-4.20.tar.gz)

Extract source ```tar -xf selinux-4.20-rc7.tar.gz```

Copy the configuration file ```selinux-4.20-rc7``` from the configs folder into the directory with the extracted source

Rename the configuration file to ```.config```

## Compile, pick one method
#### Build on chromebook
Requires a few library dependencies.
``` bash
make -j4
sudo make install
sudo make install dtbs
sudo make modules_install
# sudo make headers_install # optional
sudo cp ./arch/arm/boot/zImage /boot
```

#### Build on Arch Linux host
Requires x-tools7h to cross compile, this should be provided by [distccd-alarm-armv7h](https://aur.archlinux.org/packages/distccd-alarm-armv7h/)

``` bash 
export ARCH=arm
# change to the path of your cross toolchain
export CROSS_COMPILE=arm-linux-gnueabihf-
export INSTALL_MOD_PATH=/tmp/kernel
export INSTALL_PATH=$INSTALL_MOD_PATH/boot
export INSTALL_HDR_PATH=$INSTALL_MOD_PATH/usr
mkdir -p $INSTALL_MOD_PATH $INSTALL_PATH $INSTALL_HDR_PATH
# change the 8 in -j8 to the number of make jobs that you want to run
make -j8 modules_install &&
make -j8 install &&
make -j8 INSTALL_HDR_PATH=$INSTALL_HDR_PATH headers_install && # This command IGNORES predefined variables
cp arch/arm/boot/zImage $INSTALL_PATH &&
cp arch/arm/boot/dts/*.dtb $INSTALL_PATH
```
This script is derived from the [RockMyy](https://github.com/Miouyouyou/RockMyy) script.

Compressing the kernel makes it easier to transfer to the device.
```tar -cvf kernel.tar /tmp/kernel```

# Setting up a Fedora rootfs
### Write partition layout
Follow the Arch Linux guide and stop after step 9: [Asus Chromebook Flip C100P](https://archlinuxarm.org/platforms/armv7/rockchip/asus-chromebook-flip-c100p)

### Write rootfs
Download the rootfs [Fedora 29 ARMFP(armv7h) minimal ](https://download.fedoraproject.org/pub/fedora/linux/releases/29/Spins/armhfp/images/Fedora-Minimal-armhfp-29-1.2-sda.raw.xz) or whatever is newest.
 
Extract the image and mount it to a loopback device.
``` bash
unxz Fedora-Minimal-armhfp-29-1.2-sda.raw.xz
losetup -f -P Fedora-Minimal-armhfp-29-1.2-sda.raw
```
The loopback device should be visible in ```lsblk```
```
loop0       7:0    0   1.8G  1 loop
├─loop0p1 259:0    0    76M  1 part
├─loop0p2 259:1    0   489M  1 part  /run/media/user/__boot
└─loop0p3 259:2    0   1.2G  1 part
```
Substitute ```/dev/sde``` with the correct disk and ```loop0``` with the correct loopback device.
``` bash
# Write rootfs
dd if=/dev/loop0p3 of=/dev/sde2
# Check and expand partition
e2fsck -f /dev/sde2
resize2fs /dev/sde2
```
Mount and delete  ```/etc/fstab```, don't need it
```
mount /dev/sde2 /mnt
rm /mnt/etc/fstab
umount /dev/sde2
```

# Flash the kernel to the kernel partition
See [Kernel_Signer](https://github.com/Dogcatfee/Kernel_Signer)
