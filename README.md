# Building a basic kernel
### Set-up and configure source
Download the kernel source [selinux-4.20-rc7.tar.gz](https://git.kernel.org/pub/scm/linux/kernel/git/pcmoore/selinux.git/snapshot/selinux-4.20.tar.gz)

Extract source ```tar -xf selinux-4.20-rc7.tar.gz```

Copy the configuration file ```selinux-4.20-rc7``` from the configs folder into the directory with the extracted source

Rename the configuration file to ```.config```

## Compile, pick one method
### Build on chromebook
Requires a few library dependencies.
``` bash
make -j4
sudo make install
sudo make install dtbs
sudo make modules_install
# sudo make headers_install # optional
sudo cp ./arch/arm/boot/zImage /boot
```

### Build on Arch Linux host
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

### Flashing the kernel
See [Kernel_Signer](https://github.com/Dogcatfee/Kernel_Signer)