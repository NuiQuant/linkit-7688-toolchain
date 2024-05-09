# linkit-smart-7688 with gs-usb support


## Build the firmware from sources

This section describes how to build OpenWRT for LinkIt Smart 7688 from source codes.
1. Install prerequisite packages for building the firmware [https://openwrt.org/docs/guide-developer/toolchain/install-buildsystem](https://openwrt.org/docs/guide-developer/toolchain/install-buildsystem):
    
    ```
    $ sudo apt update
    $ sudo apt install build-essential clang flex bison g++ gawk \
    $ gcc-multilib g++-multilib gettext git libncurses-dev libssl-dev \
    $ python3-distutils rsync unzip zlib1g-dev file wget
    ```
2. Download and update the sources, select a specific code revision:
    
    ```
    $ git clone https://git.openwrt.org/openwrt/openwrt.git
    $ cd openwrt
    $ git pull
    $ git branch -a
    $ git tag
    $ git checkout v23.05.3
    ```
3. Prepare the default configuration file for feeds:
    
    ```
    $ cp feeds.conf.default feeds.conf
    ```
4. Add the LinkIt Smart 7688 feed [https://github.com/MediaTek-Labs/linkit-smart-7688-feed](https://github.com/MediaTek-Labs/linkit-smart-7688-feed):
    
    ```
    $ echo src-git linkit https://github.com/MediaTek-Labs/linkit-smart-7688-feed.git >> feeds.conf
    ```
5. Update the feeds:
    
    ```
    $ ./scripts/feeds update -a
    $ ./scripts/feeds install -a
    ```
6. To `package/kernel/linux/modules/can.mk` add lines:
    
    ```
    define KernelPackage/can-gs-usb
      TITLE:=Geschwister Schneider UG CAN/USB interface
      KCONFIG:=CONFIG_CAN_GS_USB
      FILES:=$(LINUX_DIR)/drivers/net/can/usb/gs_usb.ko
      AUTOLOAD:=$(call AutoProbe,gs_usb)
      $(call AddDepends/can,+kmod-usb-core)
    endef
    
    define KernelPackage/can-gs-usb/description
     This driver supports the Geschwister Schneider and bytewerk.org
     candleLight USB CAN interfaces USB/CAN devices
    endef
    
    $(eval $(call KernelPackage,can-gs-usb))
    ```
7. Configure the firmware image:
    
    ```
    $ make menuconfig
    ```
    * Select the options:
        * Target System: `MediaTek Ralink MIPS`
        * Subtarget: `MT76x8 based boards`
        * Target Profile: `MediaTek LinkIt Smart 7688`
        * Kernel modules:
            * CAN Support:
                * kmod-can
                * kmod-can-gs-usb
                * kmod-can-raw
        * Network
            * Routing and Redirection
                * ip-full
        * Other modules/utilites/languages if needed (usbutils, canutils-cansend, canutils-candump, luci, python)
    * Save and exit (**use the deafult config file name without changing it**)
8. Start the compilation process:
    
    ```
    $ make -j$(nproc) defconfig download clean world
    ```
9. After the build process completes, the resulted firmware file will be under `bin/targets/ramips/mt76x8/openwrt-ramips-mt76x8-mediatek_linkit-smart-7688-squashfs-sysupgrade.bin`. You can use this file to do the firmware upgrade through the Web UI. Or rename it to `lks7688.img` for upgrading through a USB drive.


## Configure network
    
    ```
    $ ip link set can0 type can bitrate 100000
    $ ifconfig can0 up
    $ ifconfig can0 down
    ```
## Links
    OpenWRT buildsystem [prereq https://openwrt.org/docs/guide-developer/toolchain/install-buildsystem](prereq https://openwrt.org/docs/guide-developer/toolchain/install-buildsystem)
    OpenWRT build [https://openwrt.org/docs/guide-developer/toolchain/use-buildsystem](https://openwrt.org/docs/guide-developer/toolchain/use-buildsystem)
    GS-USB patch (not needed on 23.05.3) [https://github.com/normaldotcom/socketcan_gs_usb](https://github.com/normaldotcom/socketcan_gs_usb{
    LinkIt Smart 7688 feeds [https://github.com/MediaTek-Labs/linkit-smart-7688-feed](https://github.com/MediaTek-Labs/linkit-smart-7688-feed)
    
