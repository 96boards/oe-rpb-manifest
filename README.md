Linaro OE RPB releases for Qualcomm Snapdragon
==============================================

This branch contains an archive of all Linaro OE RPB releases for Qualcomm Snapdragon processors. The releases are made using [repo - The Multiple Git Repository Tool](https://gerrit.googlesource.com/git-repo/). Each release has a corresponding repo manifest file (XML), located in this repository.

The OE RPB buildsystem is using various components from the Yocto Project, most importantly the Openembedded buildsystem, the bitbake task executor and various application and BSP layers.

To configure the scripts and download the build metadata, do:
```
mkdir ~/bin
PATH=~/bin:$PATH

curl http://commondatastorage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
```
Run repo init to bring down the latest version of Repo with all its most recent bug fixes. You need to identify the XML file corresponding to the release that needs to be checked out, since it is a parameter for the repo init command. 
```
repo init -u https://github.com/96boards/oe-rpb-manifest.git -b qcom/release -m <BOARD/RELEASE.xml>
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

Setup Environment
-----------------

It is recommended to check the corresponding releases notes for detailed instructions about how to setup, configure and build the release.

Support
-------

If you find any bugs please report them here

https://github.com/96boards/oe-rpb-manifest/issues

If you have questions or feedback, please subscribe to

https://lists.linaro.org/mailman/listinfo/openembedded
