---
layout: page
title: HOWTO
---

This page contains information about using Linux for Vehicle-to-X
communication, also known as Intelligent Transportation Systems or
IEEE 802.11p.

* This will become a table of contents (this text will be scraped).
{:toc}

News
------------

2015-12-09
: Bernd Lehmann will talk at
[32nd Chaos Communication Congress (32C3)](https://events.ccc.de/congress/2015/wiki/Main_Page)
about [Vehicle2Vehicle Communication based on IEEE 802.11p](http://halfnarp.events.ccc.de/).

2015-11-23
: Patches for supporting 5.9 GHz band [posted](https://www.mail-archive.com/ath9k-devel@lists.ath9k.org/msg13886.html).

2015-09-29
: Patch adding OCB mode support to ath9k driver
  [will appear in Linux 4.4](http://mid.gmane.org/20150929074312.5DB17140D97@smtp.codeaurora.org).


Installation
------------

### Linux kernel

Mainline Linux kernel 3.19 contains most of the changes needed
for 802.11p support. The only missing part is the patch for ath9k
driver, which needs to be fetched from our
[repository](https://github.com/CTU-IIG/802.11p-linux.git).

Clone the repository (this will take a *few minutes*)

    git clone --branch its-g5_v3 https://github.com/CTU-IIG/802.11p-linux.git
    cd 802.11p-linux

Install needed packages if you haven't them already

    sudo apt-get install gcc libncurses5-dev make

Prepare the folder for compilation

    mkdir _build

Use the appropriate defconfig

    make O=_build x86_64_defconfig

Configure the kernel if necessary (enable ATH_9K, MAC80211_OCB_DEBUG,
CONFIG_MAC80211_STA_DEBUG etc.)

    cd _build
    make menuconfig

Run the compilation

    make -j4

Install the modules and the kernel

    sudo make modules_install
    sudo make install


### iw -- wireless configuration tool

IEEE802.11p support is
[available in `iw` 4.0 and later](https://git.kernel.org/cgit/linux/kernel/git/jberg/iw.git/commit/?id=3955e5247806b94261ed2fc6d34c54e6cdee6676).
If your system has an older version follow this procedure.

Install pkg-config and libnl development files

    sudo apt-get install pkg-config libnl-genl-3-dev

Clone the
[official iw repository](http://git.kernel.org/cgit/linux/kernel/git/jberg/iw.git)

    git clone git://git.kernel.org/pub/scm/linux/kernel/git/jberg/iw.git
    cd iw

Build it

    make

Install it

    sudo PREFIX=/ make install

Test it

    /sbin/iw | grep -i ocb
     	dev <devname> ocb leave
     	dev <devname> ocb join <freq in MHz> <5MHZ|10MHZ> [fixed-freq]


### wireless-regdb -- regulatory information

Install the needed dependencies

    sudo apt-get install python-m2crypto

Clone the repository, compile it, install it

    git clone --branch its-g5_v1 https://github.com/CTU-IIG/802.11p-wireless-regdb.git
    cd 802.11p-wireless-regdb
    make
    sudo make install PREFIX=/


### CRDA -- Central Regulatory Domain Agent

Install some extra packages

    sudo apt-get install python-m2crypto libgcrypt11-dev

Clone the repository

    git clone --branch its-g5_v1 https://github.com/CTU-IIG/802.11p-crda.git
    cd 802.11p-crda

Copy your public key (installed by wireless-regdb, see above) to CRDA sources

    cp /lib/crda/pubkeys/$USER.key.pub.pem pubkeys/

Compile and install CRDA (custom REG_BIN is needed on Debian)

    make REG_BIN=/lib/crda/regulatory.bin
    sudo make install PREFIX=/ REG_BIN=/lib/crda/regulatory.bin

Test CRDA and the generated regulatory.bin

    sudo /sbin/regdbdump /lib/crda/regulatory.bin | grep -i ocb
    country 00: invalid
     	(5850.000 - 5925.000 @ 20.000), (20.00), NO-CCK, OCB-ONLY


Now is the right time to reboot the computer into the newly compiled
kernel.


Configuring a wireless interface for OCB mode
-------------------------------------------

    sudo iw reg set DE
    sudo ip link set wlan0 down
    sudo iw dev wlan0 set type ocb
    sudo ip link set wlan0 up
    sudo iw dev wlan0 ocb join 5890 10MHZ

Get the interface statistics

    ip -s link show dev wlan0
