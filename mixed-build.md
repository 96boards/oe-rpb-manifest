Using bitbake_secondary_image to create mixed 32bit/64bit builds
================================================================

OE RPB has support for creating mixed 32-bit/64-bit builds with 64-bit
kernel and 32-bit userland for:
* HiKey
* Dragonboard-410c

There are two variants of machine configuration for HiKey and Dragonboard-410c
boards:

| Board |  MACHINE-32   |  MACHINE-64 |
|:-----:|:-------------:|:-----------:|
| HiKey | hikey-32 | hikey |
| DB-410c| dragonboard-410c-32 | dragonboard-410c |

MACHINE-32 configuration doesn't build the kernel. It is intended to
create the 32-bit root filesystem only.

MACHINE-64 configuration is universal. But in this mixed build only the
kernel and the kernel modules are needed from the 64-bit configuration,
so the 64-bit rpb-minimal-image is built.

Initial Configuration
---------------------

One should follow the [generic instructions](README.md) to set up Repo
and to download the metadata sources - up to the "Setup Environment" step.

Setup Environment
-----------------
Run
```
$ . setup-environment
```
and select `<MACHINE-32>` as the MACHINE.

DISTRO values can be:
* rpb-x11
* rpb-wayland

Then do
```
bitbake_secondary_image --extra-machine <MACHINE-64> <image>rpb-weston-image
```
e.g. `bitbake_secondary_image --extra-machine dragonboard-410c rpb-weston-image`
(provided that MACHINE=dragonboard-410c-32 and DISTRO=rpb-wayland were selected
when sourcing setup-environment)

Creating the images
-------------------
`bitbake_secondary_image` actually runs two builds. So in the build directory,
under `tmp-*/deploy/images/` two directories are created: one for 32-bit build
artifacts, and the other for the 64-bit ones. E.g.
```
tmp-rpb_wayland-glibc/deploy/images/dragonboard-410c-32
```
and
```
tmp-rpb_wayland-glibc/deploy/images/dragonboard-410c
```
Unpack the 32-bit `*.rootfs.ext4` image, resize it to make sure that there is
enough space for the 64-bit modules, mount it via a loop device, and unpack the
64-bit modules into the 32-bit root filesystem. Then unmount the rootfs to get
the 32-bit rootfs.ext4 image with the 64-bit kernel modules added.

E.g. for Dragonboard-410c (assuming that all the relevant build artifacts
are in the current directory):
```
gunzip -k rpb-weston-image-dragonboard-410c-32-20161013104111.rootfs.ext4.gz
resize2fs rpb-weston-image-dragonboard-410c-32-20161013104111.rootfs.ext4 512M
mkdir root
sudo mount -o loop rpb-weston-image-dragonboard-410c-32-20161013104111.rootfs.ext4 root
cd root/
sudo tar xzf ../modules--4.4-r0-dragonboard-410c-20161013094521.tgz
cd ..
sync
sudo umount root
```

For HiKey in addition to the 64-bit moodules, the 64-bit kernel image and the DTB file must be copied to the 32-bit rootfs, the /boot directory (this is where
GRUB looks the kernel image for) . E.g.:
```
mkdir root
mkdir root-64
gunzip -k rpb-minimal-image-hikey-20161014162659.rootfs.ext4.gz
sudo mount -o loop rpb-minimal-image-hikey-20161014162659.rootfs.ext4 roo
t-64/
gunzip -k rpb-weston-image-hikey-32-20161014172406.rootfs.ext4.gz 
resize2fs rpb-weston-image-hikey-32-20161014172406.rootfs.ext4 512M
sudo mount -o loop rpb-weston-image-hikey-32-20161014172406.rootfs.ext4 root
sudo cp -r root-64/boot/* root/boot/
cd root
sudo tar xzf ../modules--4.4.11+git-r0-hikey-20161014162659.tgz
cd ..
sync
sudo umount root
sudo umount root-64
```

Flashing the images
-------------------
To flash the image(s) into the board follow the standard instructions for
the board. The only difference is that for the rootfs the just created mixed
rootfs.ext4 image is to be used.
