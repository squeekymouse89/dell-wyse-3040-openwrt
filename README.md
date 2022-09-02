# Dell Wyze 3040

This is mainly about building and installing OpenWRT on the Wyse 3040. This is an altered fork that included virtualisation, qemu (wuth usb passthrough) and USB external storage support.
## TOC

- [Dell Wyze 3040](#dell-wyze-3040)
  * [Notes](#notes)
    + [Dell Pages](#dell-pages)
    + [Articles](#articles)
  * [Hardware](#hardware)
  * [Boot Options](#boot-options)
  * [BIOS Updates](#bios-updates)
  * [BIOS Setup](#bios-setup)
  * [Operating Systems](#operating-systems)
    + [Linux](#linux)
      - [OpenWRT](#openwrt)
        * [Building](#building)
        * [My Unofficial Builds](#my-unofficial-builds)
      - [ThinLinux](#thinlinux)
      - [Mint](#mint)
      - [antiX Linux](#antix-linux)
      - [VyOS](#vyos)
  * [Images](#images)
    + [Device](#device)
    + [OpenWRT](#openwrt-1)
    + [antiX](#antix)


## Notes

### Dell Pages

* [Wyse 3040 Thin Client](https://www.dell.com/en-us/work/shop/wyse-endpoints-and-software/wyse-3040-thin-client/spd/wyse-3040-thin-client)
* [Wyse 3040 Disassembly and Reassembly](https://www.dell.com/support/manuals/en-us/wyse-3040-thin-client/3040_ug/disassembly-and-reassembly?guid=guid-2832a3ba-4312-4770-98e8-dd0261ca350c&lang=en-us)
* [Wyse 3040 Client User Guide](https://www.dell.com/support/manuals/en-us/wyse-3040-thin-client/3040_ug/system-specifications?guid=guid-b35dd1df-32f3-4c36-84a9-52d9a5c0810c&lang=en-us)

### Articles

* [Thin Clients Wyse 3040 (N10D): Hardware](https://www.parkytowers.me.uk/thin/wyse/3040/)
* [Thin Clients Wyse 3040 Disassembly](https://www.parkytowers.me.uk/thin/wyse/3040/disassembly.shtml)
* [A Baby WYSE, the 3040](https://blog.kroy.io/2020/01/17/the-baby-wyse-the-dell-3040/)
* [Install a New OS On a Dell Wyse 3040](https://qubitsandbytes.co.uk/install-a-new-os-on-a-dell-wyse-3040/)

## Hardware

* **Power:** My device shows 5V @ 3A, I use [this power adapter](https://www.amazon.com/dp/B0877ZTXT2?ref=ppx_yo2ov_dt_b_product_details).
    * [Video Power Mod, Teardown, UEFI/BIOS Quirks](https://www.youtube.com/watch?v=6Ls7xn4qdlk).
* **CPU:** [Intel(R) Atom(TM) x5-Z8350 CPU @ 1.44GHz](https://ark.intel.com/content/www/us/en/ark/products/93361/intel-atom-x5z8350-processor-2m-cache-up-to-1-92-ghz.html)
* **Memory:** 2GB DDR3 1600 MHz Soldered
* **Drive:** 8GB EMMC SK Hynix H56C4HP4A - There is also a 16GB version of this board.
* **Audio:** PulseAudio shows:
    * Intel HDMI/DP LPE Audio - snd_hdmi_lpe_audio
    * Dell Inc. - Wyse 3040 ThinClient -- CherryTrailCR - snd_soc_sst_cht_bsw_rt5672
* **USB:** 
    * 3x USB 2.0
    * 1x USB 3.1
* **Video:** 
    * 2x DisplayPort 
    * Max Resolution 1920x1080
    * Intel Corporation Atom Processor x5-E8000 Integrated Graphics Controller
* **Ethernet:** 1x Ethernet 1 Gbps - Realtek RTL8111/8168/8411
* **Expansion:** M.2 E-Key
    * This is NOT a true E-Key slot, it is an *SDIO interface*.
    * There is more SDIO wifi info in this [FreeBSD article](https://wiki.freebsd.org/SDIO).
    * I tried various E-Key wifi cards, none worked.
    * Reference this `lspci` line:

            00:11.0 SD Host controller: Intel Corporation 
                    Atom/Celeron/Pentium Processor x5-E8000/J3xxx/N3xxx 
                    Series SDIO Controller (rev 36)
* **lshw - Hardware** [lshw.txt](https://raw.githubusercontent.com/pjobson/openwrt-dell-wyze-3040/main/notes/lshw.txt?token=GHSAT0AAAAAABTRQNXQO5Y66VO6U6DAEJSKYUW4K5A)
* **lspci - PCI** [lspci.txt](https://raw.githubusercontent.com/pjobson/openwrt-dell-wyze-3040/main/notes/lspci.txt?token=GHSAT0AAAAAABTRQNXQLILXPEOHE2AN37DQYUW4LWA)
* **pacmd - Audio** [pacmd.txt](https://raw.githubusercontent.com/pjobson/openwrt-dell-wyze-3040/main/notes/pacmd.txt?token=GHSAT0AAAAAABTRQNXQB4UFOPRH6KLZF7SUYUXAXAA)

## Boot Options

* F2 - Boots BIOS
* F12 - Boot Menu

## BIOS Updates

My unit would not let me update to 1.2.5 from 1.2.4, no clue why. It just showed `<invalid>` in the updater.

* Download from Dell: [https://www.dell.com/support/home/en-us/product-support/product/wyse-3040-thin-client/drivers](https://www.dell.com/support/home/en-us/product-support/product/wyse-3040-thin-client/drivers)
* Unzip it if it is zipped.
* Copy to a USB flash drive, FAT32 formatted.
* Boot the with F12 key.
* Follow instructions on screen.

## BIOS Setup

* Enter BIOS with F12 
* System Configuration -> USB Configuration, then check **Enable USB Boot Support**
* Secure Boot -> Secure Boot Enaable, set to **Disabled**
* Power Management -> AC Recovery, set to **Power On**
* Save Settings

## Operating Systems

### Linux

#### OpenWRT

The current build of OpenWRT does not install on the Wyze.  From what I understand, this is due to the default build outputting to Serial.  I built my own version specifiying the Atom processor with no serial output and had some good luck.

##### Building

I'm not an expert on building OpenWRT, this is the first custom build I've done.  I may be missing some stuff or could be wrong about any amount of this.  Feel free to drop a comment/bug/pull request.


**Install for Ubuntu 22.04.1 LTS Server:**

sudo apt update
sudo apt install build-essential gawk gcc-multilib flex git gettext libncurses5-dev libssl-dev python3-distutils rsync unzip zlib1g-dev file
rm -rf openwrt
git clone https://github.com/openwrt/openwrt
#cd openwrt
git pull
# list tags
#git tag
# I'm using the latest rc as of today
git checkout v21.02.3

# Update/install the feeds
./scripts/feeds update -a
./scripts/feeds install -a

# Download my config
wget "https://raw.githubusercontent.com/squeekymouse89/dell-wyse-3040-openwrt/main/config/config" -O .config

# Otherwise make your own
# Make your config
cd openwrt
make menuconfig
# Make your kernel config
make kernel_menuconfig

# Do the build, it'll take like an hour.
make world -j$(nproc) V=sc

# You can find the builds in: openwrt/bin/
# This includes all the packages.
find . -name "openwrt-x86-64-generic-ext4-combined-efi.img.gz"

```

##### My Unofficial Builds

* [openwrt.wyse.withqemu.img.gz](https://github.com/squeekymouse89/dell-wyse-3040-openwrt/blob/main/Builds/openwrt.wyse.withqemu.zip?raw=true)

**Installing:**

I used a windows 10 boot cd and balena etcher on a usb stick to image my wyse machine. I used rufus to create a windows 10 install then install balana enter to the same stick.

I then simply used Shift F10 to open a cmd prompt in setup and execute etcher to flash my device.
