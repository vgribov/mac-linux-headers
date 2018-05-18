# README

On OS X we don't have some headers from a Linux world which seems to be
essential for building a Linux kernel. This small project aims to provide the
missing headers and a short readme for those who (like me) wants to work with
Linux on Mac as native as possible.

For the beginning you should install `libefi` with

    brew install libefi

Clone the repository and copy its files into `/usr/local/include`.

Then install some GNU stuff to make feel kernel's Makefiles at home:

    brew install --with-default-names coreutils libtool gnu-sed grep

To make Linux ELF binaries you'll need a cross compiler. I found a nice one at
<https://github.com/richfelker/musl-cross-make>.

Now you should be able to build the kernel with

    make x86_64_defconfig
    make CROSS_COMPILE='x86_64-linux-musl-'

To be able to use a `menuconfig` you should also install `gettext` and
`ncurses`:

    brew install gettext ncurses
    brew link --force gettext
    brew link --force ncurses
