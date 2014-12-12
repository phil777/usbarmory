FROM debian:wheezy
ENV DEBIAN_FRONTEND noninteractive
WORKDIR /ua

## Add i386 arch for linaro toolchain
RUN dpkg --add-architecture i386
RUN apt-get update
RUN apt-get -y install bc parted debootstrap binfmt-support qemu-user-static uboot-mkimage kmod git make gcc libc6:i386 libstdc++6:i386 zlib1g:i386 

## Install Linaro toolchain
RUN wget http://releases.linaro.org/14.09/components/toolchain/binaries/gcc-linaro-arm-none-eabi-4.9-2014.09_linux.tar.xz
RUN tar xf gcc-linaro-arm-none-eabi-4.9-2014.09_linux.tar.xz

## Download and build u-boot for USB armory
RUN git clone https://github.com/inversepath/u-boot-usbarmory.git
RUN cd u-boot-usbarmory ; \
    make distclean ; \
    make usbarmory_config ; \
    make ARCH=arm CROSS_COMPILE=/ua/gcc-linaro-arm-none-eabi-4.9-2014.09_linux/bin/arm-none-eabi-


## Download and build kernel
RUN wget http://ftp.kernel.org/pub/linux/kernel/v3.x/linux-3.16.2.tar.gz
RUN tar zxf linux-3.16.2.tar.gz
ADD linux.config /ua/linux-3.16.2/.config
RUN cd linux-3.16.2 ; make ARCH=arm CROSS_COMPILE=/ua/gcc-linaro-arm-none-eabi-4.9-2014.09_linux/bin/arm-none-eabi- uImage LOADADDR=0x70008000 
RUN cd linux-3.16.2 ; make ARCH=arm CROSS_COMPILE=/ua/gcc-linaro-arm-none-eabi-4.9-2014.09_linux/bin/arm-none-eabi- dtbs 
RUN cd linux-3.16.2 ; make ARCH=arm CROSS_COMPILE=/ua/gcc-linaro-arm-none-eabi-4.9-2014.09_linux/bin/arm-none-eabi- modules

## Prepare a debian root
# work arround unability to mount /proc in docker build mode
RUN dpkg-divert --local  --add /usr/share/debootstrap/functions
ADD debootstrap.functions /usr/share/debootstrap/functions

RUN qemu-debootstrap --arch=armhf --include=ssh wheezy /armroot http://ftp.debian.org/debian/
RUN rm /armroot/var/log/bootstrap.log
RUN chroot /armroot apt-get clean

ADD inittab /armroot/etc/inittab
ADD rc.local /armroot/etc/rc.local
ADD hostname /armroot/etc/hostname
ADD resolv.conf /armroot/etc/resolv.conf
ADD sources.list /armroot/etc/apt/sources.list

## Add installer script
ADD install install

CMD bash


