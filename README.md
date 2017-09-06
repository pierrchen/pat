# Poplar Android Flash tools

It will generate the install scripts and the images that can be put into a FAT32 formatted usb disk, and flashed to the board emmc through u-boot command.

## Dependency

1. Ruby

This is a ruby script so need to install ruby if have don't have one.
On Ubuntu ` sudo apt-get install ruby-ful`

2. simg2img

Used by the script to convert Android sparse image to raw ext4 image. This will be available after a full android build, located at `${your_android}/out/host/linux-x86/bin/simg2img`

3. A FAT32 formatted usb with capacity bigger than 8G. To format, check [this](https://askubuntu.com/questions/22381/how-to-format-a-usb-flash-drive).

## Instructions

1. Download and build the poplar Android (follow internal wiki)

2. Check the pat tool to anywhere you want. `git clone git@github.com:pierrchen/pat.git /path/to/pat_tool`; and move `cp $OUT/*.img path/to/pat_tool`. 

(The `$OUT` is an environment variable setting by android build system after you did `source build/envsetup.sh; lunch`. For poplar, it is `${your_android_src_root}/out/target/product/poplar`.) 

3. Run `ruby ./poplar_android_flash.rb` in pat_tool.

4. Copy following stuff (using `install_usb.sh` script as we'll show later)to your FAT32 formatted USB disk for a whole re-flash. 

```
    a) *.scr
    b) mbr.tgz and ebr*.tgz
    c) bootloader.img (*)
    d) boot.img, system.img.raw_*, usrdata.img.raw_* , cache.img.raw_*
```

Notes:

- a) are install scripts for u-boot
- b) are partition table; prebuilt, come with this repo
- c) bootloader.img; a prebuild is provided for the convenience. But you are suggested to build your own using the latest l-loader/atf/u-boot to stay in sync. The `bootloader.img` is generated from of the `l-loader.bin` by removing its first 512 bytes, as shown below. To build `l-loader.bin`, follow the instructions instructed [here](https://github.com/Linaro/poplar-tools/blob/latest/build_instructions.md)
A prebuilt is provided as well.

```
    dd if=l-loader.bin of=bootloader.img bs=512 skip=1 count=8191
```

- d) are converted from android core images, splitted, and raw or ext4 (NOT sparse image)

4. To flash all the partitions, first run `./install_usb.sh usb_mount_point` to copy all the required images and scripts to your usb disk. The usb_mount_point parameters passed in , for Ubuntu 14.04 it is `/media/<user>/<usb_id>`.

Power on the board, enter into u-boot console, copy paste following commands:

```
usb reset;fatload usb 0:1 ${scriptaddr} flash_pt.scr; source ${scriptaddr};fatload usb 0:1 ${scriptaddr} flash_bootloader.scr; source ${scriptaddr};fatload usb 0:1 ${scriptaddr} flash_system.scr; source ${scriptaddr};fatload usb 0:1 ${scriptaddr} flash_userdata.scr; source ${scriptaddr};fatload usb 0:1 ${scriptaddr} flash_cache.scr; source ${scriptaddr};
```
