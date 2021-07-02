# Raspberry Pi - Boot2Image
Bash script to generate custom bootable OS img files for Raspberry Pi

## Dependencies
The script makes use of the following tools :

    du
    dd
    losetup
    fdisk
    mkfs.vfat

Normally these are available in all distributions by default, but if anyone of them is missing please refer to the instruction manuals of your particular distribution.

## Setting up
### Downloading the script
Download the script (preferably in a empty folder) using the following commands :
Using wget :

    wget https://raw.githubusercontent.com/oddlyspaced/rpi-boot2img/main/generate-img.sh
Alternatively using cURL :

    curl https://raw.githubusercontent.com/oddlyspaced/rpi-boot2img/main/generate-img.sh --output generate-img.sh

### Setting executable permission
Once the script has been downloaded, make sure to set executable permissions on it by using the following command :

    chmod +x generate-img.sh


### Editing variables in script
Open the `generate-img.sh` file in a text editor and you'll notice a couple of different variables present in the top part of the script
The following variables are required to be edited by the user accordingly

    # source files for boot partition (these files will be copied over to the image)
    source_files=/home/hardik/Raspberry-Pi/mount-bkp/*
    # final img file name
    final_img=generated.img
The `source_files` variable points to a folder which contains the files you want to put in the boot partition.
The `final_img` variable will define the name of the generated custom img file.

Other global variables which the script uses exist in the section called `Script variables` and one can edit them accordingly if needed.


## Running (Generating the img file)
To generate the custom img file, just execute

    ./generate-img.sh

or

    bash generate-img.sh

and it should generate the img file in the same directory successfully.

> Do note that the script has several sudo commands in plance and they
> would require user authentication, so make sure to enter the password
> when prompted.

## Example
I originally developed this script to generate custom OS images while learning bare metal development.
For Raspberry Pi 4B the `source_files` targets my folder which has the path `/home/hardik/Raspberry-Pi/mount-bkp/*` and contains the following files :
    
    ├── bcm2711-rpi-4-b.dtb
    ├── bootcode.bin
    ├── cmdline.txt
    ├── config.txt
    ├── kernel8-rpi4.img
    ├── overlays
    │   ├── miniuart-bt.dtbo
    │   └── overlay_map.dtb
    └── start4.elf
    
    1 directory, 8 files

Keep note of the `*` in the source_files path. This essentially tells the script to copy over the files inside the folder. Omitting the `*` can copy the whole folder and can possibly result in a non functioning img file.

Other variables can be left as it is, and on executing `./generate-img.sh` an img file is successfully created which is of `200 MB` and has the following partition table :

    Disk generated.img: 200 MiB, 209715200 bytes, 409600 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disklabel type: dos
    Disk identifier: 0xc0844a25
    
    Device         Boot Start    End Sectors  Size Id Type
    generated.img1       2048 409599  407552  199M  c W95 FAT32 (LBA)

## Extras
### Flashing the generated img
The generated image file can be flashed using the official Raspberry Pi Imager or via dd using the following command :

    sudo dd if=generated.img of=<device path> bs=4M
where `<device path>` is the path of your SD card interface (Example: `/dev/sdb`)

### Mounting the boot in generated img
Since the generated img contains only 1 partition, you can directly mount it using the following command :

    sudo mount -v -o offset=1048576 -t vfat generated.img mnt

Where `1048576` is obtained by multiplying the start offset with 512. [`512 * 2048`]

Alternatively you can setup a loop device and mount the partition via that by running :

    sudo losetup $(losetup -f) generated.img
    sudo mount /dev/loop?p? /path/to/mount/point

Here `?`  in `/dev/loop?p?` would refer to the loop device on which the generated.img was mounted `(Example /dev/loop0p1)` and `/path/to/mount/point` is the mount point
