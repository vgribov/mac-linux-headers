# README

On OS X we don't have some headers from a Linux world which seems to be
essential for building a Linux kernel. This small project aims to provide the
missing headers and a short readme for those who (like me) wants to work with
Linux on Mac as native as possible.

## Building a Linux kernel

1. Install `libelf` with

```sh
brew install libelf
```

2. Clone the repository and copy its files into `/usr/local/include`:

```sh
git clone git@github.com:vgribov/mac-linux-headers.git
cp -r mac-linux-headers/* /usr/local/include
```

3. Install some GNU stuff to make feel kernel's Makefiles at home:

```sh
brew install --with-default-names coreutils gnu-sed grep
```

To make Linux ELF binaries you'll need a cross compiler. I found a nice one at
<https://github.com/richfelker/musl-cross-make>.

Now you should be able to build the kernel with

```sh
make x86_64_defconfig
make CROSS_COMPILE='x86_64-linux-musl-'
```

To be able to use a `menuconfig` you should also install `gettext` and
`ncurses` and link them:

```sh
brew install gettext ncurses
brew link --force gettext
brew link --force ncurses
```

## Build and run a minimal Linux in Qemu

This part is based on <https://gitlab.eurecom.fr/snippets/23>, so all credits
go to <mailto:renaud.pacalet@telecom-paristech.fr>.

1. Download and unpack your preferred Linux kernel version:

```sh
wget https://cdn.kernel.org/pub/linux/kernel/v4.x/linux-4.1.51.tar.xz
tar xvf linux-4.1.51.tar.xz
```

2. Prepare a basic Linux kernel configuration:

```sh
cd linux-4.1.51
mkdir build
make O=./build allnoconfig
cd ./build
make menuconfig
```

3. Configure the kernel according to the following:

```
64-bit kernel ---> yes
General setup ---> Initial RAM filesystem and RAM disk (initramfs/initrd) support ---> yes
General setup ---> Configure standard kernel features ---> Enable support for printk ---> yes
Executable file formats / Emulations ---> Kernel support for ELF binaries ---> yes
Executable file formats / Emulations ---> Kernel support for scripts starting with #! ---> yes
Device Drivers ---> Generic Driver Options ---> Maintain a devtmpfs filesystem to mount at /dev ---> yes
Device Drivers ---> Generic Driver Options ---> Automount devtmpfs at /dev, after the kernel mounted the rootfs ---> yes
Device Drivers ---> Character devices ---> Enable TTY ---> yes
Device Drivers ---> Character devices ---> Serial drivers ---> 8250/16550 and compatible serial support ---> yes
Device Drivers ---> Character devices ---> Serial drivers ---> Console on 8250/16550 and compatible serial port ---> yes
File systems ---> Pseudo filesystems ---> /proc file system support ---> yes
File systems ---> Pseudo filesystems ---> sysfs file system support ---> yes
```

4. Build the kernel:

```sh
make -j8
cd ..
```

5. Download and unpack your preferred Busybox version:

```sh
wget http://busybox.net/downloads/busybox-1.28.3.tar.bz2
tar xvf busybox-1.28.3.tar.bz2
```

6. Prepare a basic Busybox configuration:

```sh
cd busybox-1.28.3
mkdir build
make O=./build defconfig
cd ./build
make menuconfig
```

7. Configure Busybox according the following:

```
Busybox Settings ---> Build Options ---> Build BusyBox as a static binary (no shared libs) ---> yes
```

8. Build and install Busybox:

```sh
make -j8
make install
cd ../..
```

9. Create an `initramfs`:

```sh
mkdir initramfs
cd initramfs
mkdir -p bin sbin etc proc sys usr/bin usr/sbin
cp -a ../busybox-1.28.3/_install/* .
```

10. Add an `init` script to the `initramfs` with the following content:

```sh
#!/bin/sh

mount -t proc none /proc
mount -t sysfs none /sys

printf "\nBoot took %s seconds\n\n" $(awk '{ print $1 }' < /proc/uptime)

exec /bin/sh
```

11. Create the `initramfs` archive:

```sh
chmod +x init
find . | cpio -ov --format=newc | gzip > ../initramfs.cpio.gz
cd ..
```

12. Install Qemu

```sh
brew install qemu
```

Now you should be able to run and test your minimal Linux with:

```sh
qemu-system-x86_64 -kernel $PWD/linux-4.1.51/build/arch/x86_64/boot/bzImage \
                   -initrd $PWD/initramfs.cpio.gz \
                   -nographic -serial mon:stdio \
                   -append 'console=ttyS0 earlyprintk=ttyS0'
```

Press `Ctrl-a x` to quit.
