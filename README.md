# Linaro OpenEmbedded Reference Platform Build (OE RPB)

The Linaro OpenEmbedded RPB project provides a reference implementation of an OpenEmbedded based distro. It contains reference images for various uses cases, such as headless systems or graphical systemd based on Xorg server or Wayland/Weston.

## Maintainers

* Koen Kooi <mailto:koen.kooi@linaro.org>
* Nicolas Dechesne <nicolas.dechesne@linaro.org>
* Fathi Boudra <mailto:fathi.boudra@linaro.org>

## Support

If you find any bugs please report them here

https://github.com/96boards/oe-rpb-manifest/issues

If you have questions or feedback, please subscribe to

https://lists.linaro.org/mailman/listinfo/openembedded

# Using OE RPB

This document provides instructions to get started with Linaro OpenEmbedded RPB project. It tries (when possible) to be generic for any board supported. The board diversity should be addressed through dedicated BSP layer then MACHINE choice.

## Introduction

This is not an introduction on OpenEmbedded or Yocto Project. If you are not familiar with OpenEmbedded and the Yocto Project, it is very much recommended to read the appropriate documentation first. For example, you can start with:
* http://openembedded.org/wiki/Main_Page
* http://yoctoproject.org/
* https://www.yoctoproject.org/documentation

In this wiki, we assume that the reader is familiar with basic concepts of OpenEmbedded.

## OE RPB Layers

|   Layer                 |   Description                    |
|:-----------------------:|:----------------------|
| oe-core | This is the main collaboration point when working on OpenEmbedded projects and is part of the core recipes. The goal of this layer is to have just enough recipes to build a basic system, this means keeping it as small as possible. |
| meta-rpb  |   This is a very small layer where the distro configurations live. Currently it houses both Reference Platform Build and Wayland Reference Platform Builds. |
| meta-oe | This layer houses many useful, but sometimes unmaintained recipes. Since the reduction in recipes to the core, meta-oe was created for everything else. There are currently approximately 650 recipes in this layer. |
| meta-browser | This layer holds the recipes for Firefox and Chromium. Both recipes require a lot of maintenance, because of this a seperate layer was created. |
| meta-qt5 | This is a cross-platform toolkit. |
| meta-linaro | This layer is used to get the Linaro toolchain. |
| meta-backports | Backport newer versions into the build which were not part of the release. |
| meta-96boards | This support layer is managed by Linaro and intended for boards that do not have their own board support layer. Currently used for the HiKey Consumer Edition board, and eventually the Bubblegum-96 board. If a vendor does not support their own layer, it can be added to this layer. |
| meta-qcom | This is the board support layer for Qualcomm boards. Currently supports IFC6410 and the DragonBoard 410c. |
| meta-st-cannes2 | This is the board support layer for ST B2260 board. |

## oe-rpb-manifest repository

These are the setup scripts for the OE RPB buildsystem. If you want to (re)build packages or images for OE RPB, this is the thing to use.
The OE RPB buildsystem is using various components from the Yocto Project, most importantly the Openembedded buildsystem, the bitbake task executor and various application and BSP layers.

To configure the scripts and download the build metadata, do:
```
mkdir ~/bin
PATH=~/bin:$PATH

curl http://commondatastorage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
```
Run repo init to bring down the latest version of Repo with all its most recent bug fixes. You must specify a URL for the manifest, which specifies where the various repositories included in the Android source will be placed within your working directory. To check out the current branch, specify it with -b:
```
repo init -u https://github.com/96boards/oe-rpb-manifest.git -b master
```
When prompted, configure Repo with your real name and email address.

A successful initialization will end with a message stating that Repo is initialized in your working directory. Your client directory should now contain a .repo directory where files such as the manifest will be kept.

To pull down the metadata sources to your working directory from the repositories as specified in the default manifest, run
```
repo sync
```
When downloading from behind a proxy (which is common in some corporate environments), it might be necessary to explicitly specify the proxy that is then used by repo:
```
export HTTP_PROXY=http://<proxy_user_id>:<proxy_password>@<proxy_server>:<proxy_port>
export HTTPS_PROXY=http://<proxy_user_id>:<proxy_password>@<proxy_server>:<proxy_port>
```
More rarely, Linux clients experience connectivity issues, getting stuck in the middle of downloads (typically during "Receiving objects"). It has been reported that tweaking the settings of the TCP/IP stack and using non-parallel commands can improve the situation. You need root access to modify the TCP setting:
```
sudo sysctl -w net.ipv4.tcp_window_scaling=0
repo sync -j1
```

## Host package Dependencies

In order to successfully set up your build environment, you will need to install the following package dependencies.

**Step 1**: You will need git installed on your Linux host machine

`$ sudo apt-get install git`

**Step 2**: Visit the OpenEmbedded (Getting Started) wiki to see which distribution specific dependencies you will need

http://www.openembedded.org/wiki/Getting_started

Setting up the build environment will first search for `whiptail`, if it is not present then it will search for `dialog`. You only need one of the following packages to ensure your setup-environement runs correctly:

`$ sudo apt-get install whiptail`

or

`$ sudo apt-get install dialog`

**Please Note**: If you are running Ubuntu 16.04 you will need to add the following line to your `/etc/apt/sources.list`

`deb http://archive.ubuntu.com/ubuntu/ xenial universe`

```shell
$ cd /etc/apt/
#vim text editor is used in this example
#sudo is used to allow editing, sources.list is set to read only
$ sudo vim sources.list
```

All required dependencies should now be installed on your host environment, you are ready to begin your build environment setup.

## Setup Environment

It is recommended to use the `setup-environment` script. Among several things, it will prompt the user for the list of supported machines and distros. The setup script will also create (if they don't exsist) the local build configuration files.

To run the setup script:

    $ . setup-environment

And follow the instructions. If you already know the value for MACHINE and DISTRO, it is possible to set them as environment variables, e.g. 

    $ MACHINE=dragonboard-410c DISTRO=rpb source setup-environment

## Build a minimal, console-only image

To build a console image, you can run:

    $ bitbake rpb-console-image

At the end of the build, your build artifacts will be found under `tmp-eglibc/deploy/images/MACHINE/`. The two main build artifacts you will use to update your board are:
* `rpb-console-image-<MACHINE>.ext4.gz` and
* `boot-<MACHINE>.img`

## Build a simple X11 image

To build an X11 image with GPU hardware accelerated support, run:

    $ bitbake rpb-desktop-image

This image includes a few additional features, such as `systemd`, `networkmanager` which makes it simpler to use. At the end of the build, the root file system image will be available as `tmp-eglibc/deploy/images/<MACHINE>/rpb-desktop-image-<MACHINE>.ext4.gz`.

Then you can finally start the X server, and run any graphical application:

    X&
    export DISPLAY=:0
    glxgears

The default X11 image includes `openbox` window manager, and it should automatically start on boot. If you want to stop (or restart) the window manager, you can use the following command:

    $ systemctl stop xserver-nodm.service
    $ systemctl start xserver-nodm.service
    
Of course, you can easily add another window manager, such as `metacity` in the image. To install `metacity` in the image, add the following to `conf/auto.conf` file:

    CORE_IMAGE_EXTRA_INSTALL += "metacity"

and rebuild the `rpb-desktop-image` image, it will now include `metacity`.

## Build a sample Wayland/Weston image

For Wayland/weston, it is needed to change the DISTRO and use `rpb-wayland` instead of `rpb`. The main reason is that in the `rpb-wayland` distro, the support for X11 is completely removed. So, in a new terminal prompt, setup a new environment and make sure to use `rpb-wayland` for DISTRO, then, you can run a sample image with:

    $ bitbake rpb-weston-image

This image includes a few additional features, such as `systemd`, `networkmanager` which makes it simpler to use. Once built, the image will be available at `tmp-eglibc/deploy/images/<MACHINE>/rpb-weston-image-<MACHINE>.ext4.gz`. 

If you boot this image on the board, you should get a command prompt on the HDMI monitor. A user called `linaro` exists (and has no password). Once logged in a VT, you run start weston with:

    weston-launch

And that should get you to the Weston desktop shell.

## Creating a local topic branch

If you need to create local branches for all repos which then can be done e.g.
```
~/bin/repo start myangstrom --all
```
Where 'myangstrom' is the name of branch you choose

## Updating the sandbox

If you need to bring changes from upstream then use following commands
```
repo sync
```
Rebase your local committed changes
```
repo rebase
```

## Guidelines / tips

### Share downloads and sstate-cache folder

When using the OpenEmbedded build, you are building the entire GNU Linux system from scratch. That requires downloading a lot of source code packages, as well as building them. When using more than one workspace or build environment, it is very much recommended to create a shared folder for the downloads, and also a shared folder for the OpenEmbedded `sstate-cache`. The `sstate-cache` contains prebuilt packages, to avoid recompiling the same package over and over again. It is essentially a specialized `ccache` for OpenEmbedded. 

The OE RPB setup scripts expects the `downloads` and `sstate-cache` folders to be located at the root of your build environment. When initializing a second build environment, and before started any build, you can simply use Linux symlinks to share these folders. 

Assuming you have already create a workspace in `$HOME/oe-rpb`, and that you are making another one in `$HOME/oe-rpb2`, then you can run the following commands:

    cd $HOME/oe-rpb2
    ln -s $HOME/oe-rpb/downloads
    ln -s $HOME/oe-rpb/sstate-cache

It is safe to use such a setup, and OpenEmbedded will work fine!

### How to clean with bitbake

TBD
 


