# OKdo Nano C100 Developer Kit 

**Disclaimer: Follow these notes at your own risk. I'm not responsible for anything you run in your devices. It's up to you!**

This page is a bunch of notes from the process I've done to set up the [OkDo C100 NVIDIA Jetson Nano 4GB Development Kit](https://www.kubii.com/en/development-kit/3882-c100-nvidia-jetson-nano-4gb-development-kit-3272496313705.html). It's a good alternative to the official [NVIDIA Jetson Nano Developer Kit](https://developer.nvidia.com/embedded/learn/get-started-jetson-nano-devkit) product which has been discontinued. The replacement product, [NVIDIA Jetson Nano Super Developr Kit](https://www.nvidia.com/en-us/autonomous-machines/embedded-systems/jetson-orin/nano-super-developer-kit/), is usually out of stock. 

The problem is that *OKdo Nano C100 Developer Kit* is an extremely poorly documented device. The [okdo website](https://www.okdo.com/) doesn't work. I really don't know if the company is still operatin, if they just don't care about the website, or if they were adquired by another company.

If you need more detail on any point and I know it, please, don't hesitate to contact me. These notes are intentionally short because they're enought for me :-)

# Hardware

* [Radxa NX4 IO Board](https://radxa.com/products/io-board/nx5-io-board/). The one I own has no *Markrom key* or *power button*.
* [NVIDIA Jetson Nano](https://developer.nvidia.com/buy-jetson?product=jetson_nano). It has a Tegra processor and the 16GB eMMC storage.

I guess that buying the [board](https://es.aliexpress.com/item/1005007130498108.html?gatewayAdapt=glo2esp)
, the [card](https://www.siliconhighwaydirect.com/ProductDetails.asp?ProductCode=900-13448-0020-000) and the 
[heatsink](https://auvidea.eu/product/heatsink-70752/) would word too. I haven't tried it!

# Powering

You can use the jack for 5V and 4A, or the microUSB for 5V and 2A. Next to the Jack, there is a jumper that determines 
the power source. With the jumper in place, the jack power will be drawn from the jack. Without the jumper, power will come from the microUSB interface.

Because the GPU's power requirements, it's strongly recommended (and probably mandatory) to use the jack.

# Booting from eMMC

Just unboxing the device, it's possible to boot directly from the eMMC. You'll find an Ubuntu 18.04.6 LTS based login screen, using `nano` as both the username and password.

Check the version with:

    nano@nano:~$ lsb_release -rcd
    Description:	Ubuntu 18.04.6 LTS
    Release:	18.04
    Codename:	bionic

It contains NVIDIA L4T ([Linux for Tegra](https://developer.nvidia.com/embedded/linux-tegra-r321)) for Tegra, a NVIDIA Linux distribution for Tegra devices. 

Check the version with:

    nano@nano:~$$ cat /etc/nv_tegra_release
    # R32 (release), REVISION: 7.2, GCID: 30192233, BOARD: t210ref, EABI: aarch64, DATE: Wed Apr 20 21:34:48 UTC 2022

Note that this is the preinstalled distribution, not the one that your are going to use. It depends on the image you install on the SD Card or USB drive.

# MicroUSB port

If you have a MicroUSB cable, you can plug it in and connect it to your computer. Your computer will detect a
new USB hub with the following components:

* A modem probably at `/dev/ttyACM0`
* A volume named LT4-README with some files containing details.
* Two network interfaces.

When the eMMC Ubuntu is running you can connect through `SSH` using `nano` as both username and password by running:

    ssh nano@192.168.55.1

To connect to the serial port, use a terminal program like `screen` or `minicom` and connect to the appropriate `/dev/tty*` device. In my case, it is:

    sudo screen /dev/ttyACM0 115200
    
Please note that the microUSB connection depends on linux running. If the linux doesn't boot, you won't have access. If that happens, you can boot from an SD card and recover it.

# Boot from SD card

Burn the image from OKdo, available from [different](https://auto.designspark.info/okdo_images/c100.img.xz) [sources](https://www.rs-online.com/designspark/okdo-software-and-downloads-hub). The file name is `c100.img`. You just need to burn into the SD card using you preferred imaging tool. After that, insert it into the device.

It's very important to connect an HDMI screen and a keyboard to see what's happening and to follow the steps to configure Ubuntu.

# Boot from USB drive

*Option 1. Through SD card*

I've tried different approaches that didn't work. Finally, I used the SD card bootloader as a gateway to boot from USB SSD by modifying `/boot/extlinux/extlinux.conf`. You just need to change the line saying `root=/dev/mmcblk0p1` to `root=/dev/sda1` or, better, `root=PARTUUID='xxxx'`. You can obtain the PARTUUID with `blkid`. It didn't work for me with `root=UUID='xxxx'`. 

This solution its enought for me. It boots fast, but I've sacrificed an SD card just solve this. Its a reasoble price to pay.

*Option 2: Creating a new boot image (TODO)*

This is a trickier option that I haven't had time to try. Theoricaly, you need to create a new image which load the differents usb drivers to boot from an USB drive. Also you can take the advantage that the `c100.img` is the same version that the one in the eMMC.

# Next steps

I have the device running with the full operating system on a SSD. Theorically, from this point on, coding should be the same that as the original NVIDIA nano... we'll see. I'll check [how Torch works](pytorch.md), the pinouts, camera and so on.  

# References

* [Ubuntu snippets](https://github.com/ferranb/ubuntu_snippets)
* https://docs.nvidia.com/jetson/archives/r34.1/DeveloperGuide/index.html#page/Tegra%2520Linux%2520Driver%2520Package%2520Development%2520Guide%2Fbootflow_jetson_nano.html%23%5B/url%5D
* [OKDO official images mirror from RS-Online](https://www.rs-online.com/designspark/okdo-software-and-downloads-hub)
* [OKDO specifications](https://docs.rs-online.com/9149/A700000009238033.pdf)
* [Customer image for SD](https://github.com/LetsOKdo/c100-bootupd)
* https://gist.github.com/KEINOS/47b0a89f32c77adbf3887b2469e4ce7c
* https://iothonpo.com/nano-c100-setup/
