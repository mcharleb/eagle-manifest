# qc-drone-mainfest
This README file contains information on building the 'meta-eagle8074'
BSP layer, and programming the images using fastboot.
Please see the corresponding sections below for details.

Table of Contents
=================

- [Dependencies](https://github.qualcomm.com/mcharleb/qc-drone-mainfest#dependencies)
- [Prerequisites](https://github.qualcomm.com/mcharleb/qc-drone-mainfest#prerequisites)
- [Patches](https://github.qualcomm.com/mcharleb/qc-drone-mainfest#patches)
- [Building the `eagle8074` BSP layer](https://github.qualcomm.com/mcharleb/qc-drone-mainfest#building-the-eagle8074-bsp-layer)
- [Programming the OS image using `fastboot`](https://github.qualcomm.com/mcharleb/qc-drone-mainfest#programming-the-os-image-using-fastboot)
- [Programming the eagle image using `fastboot`](https://github.qualcomm.com/mcharleb/qc-drone-mainfest#programming-the-eagle-image-using-fastboot)
- Notes and tips
    * [Speed up your repo sync](https://github.qualcomm.com/mcharleb/qc-drone-mainfest#speed-up-your-repo-sync)


Dependencies
============

This layer depends on:

    URI: git://git.yoctoproject.org/poky
    layers: meta
    layers: meta-yocto
    layers: meta-oe
    branch: jethro

Prerequisites
=============

Workstation used for build must be a linux x86-64 with Ubuntu 14.04 or 16.04.
Workstation must have a gcc 4.8 or latest with the support for both 64bit
and 32bit c/c++ support.

Following packages are required to be installed on the development machine. (FIXME - check this)

```
sudo apt-get install libsdl1.2-dev texinfo chrpath gawk
```

Patches
=======

Please submit any patches against this BSP to me
(mcharleb@qti.qualcomm.com)


Building the `eagle8074` BSP layer
============================================

Get the open source layers:

    $ mkdir eagle
    $ cd eagle
    $ repo init -u git@github.qualcomm.com:mcharleb/qc-drone-mainfest.git -m eagle.xml
    $ repo sync -j 16 -c --no-tags

NOTE: --depth=1 does not work

Get the SDK add-on:
    $ wget http://mcharleb3-linux.qualcomm.com:8080/qcom_flight_controller_hexagon_sdk_add_on.zip

Get the prebuilt files from QDN (TBD):

    $ unzip prebuilt_eagle8074-0.0.1.zip

You should then be able to build the partitions for Eagle:

    $ source meta-eagle8074/scripts/set_bb_env.sh
    $ bitbake cache-image persist-image userdata-image

The factory-image and recovery-image images are not yet complete

Programming the OS image using `fastboot`
=========================================

    $ fastboot erase boot
    $ fastboot flash boot build/tmp-glibc/deploy/images/eagle8074/out/boot-eagle8074.img
    $ fastboot erase userdata
    $ fastboot flash userdata build/tmp-glibc/deploy/images/eagle8074/out/userdata-eagle8074.img

Programming the eagle image using `fastboot`
===============================================

When you bake the eagle-image, assuming you have meta-qcom-drones, in build/tmp/deploy/images/eagle8074, you will find `flash_build.py`. Run that script with the device connected to fastboot:

    $ python flash_build.py
    
This flashes all the partitions from the original metabuild, along with the partitions built by eagle-image. By default, `flash_build.py` will flash UFS images in cases where the eMMC and UFS images are different. This is a good default for Dragonboard and SBC820. In case you have an eMMC based device, you may run:

    $ python flash_build.py emmc
    
Notes and tips
===============

### Speed up your repo sync

A `repo sync` walks the list of git repositories in the manifest file and performs a fetch and checkout. Given the QuIC server hosted repositories tend to get bigger due to the meta-data associated with a repositories, we can speed up partially by dropping off one typical data element representing the tags. Git tags are usefull to identify a snapshot of file versions by name. Given manifest file doesn't call for any tags, following command is useful:

    $ repo sync -j 16 -c --no-tags

Note if you observe any failure, it may be due the fact manifest contains a project with a revision identified by tag. You will notice the failure during that project's checkout. A normal `repo sync` will subsequently update your projects with tags included.

TODO : Further improvement can be achieved if the git fetch is limited to the branch of interest (single branch and shallow history)

