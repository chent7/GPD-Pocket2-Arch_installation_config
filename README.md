# Arch Linux on the GPD Pocket 2 (WIP).                                                                                                       
                                                                                 (.                    
                                                                                (((.                   
                                                                               (((((                   
            .,,,,,  .,,,,,,     .,,,,,                                        ////((/                  
        .#%%####%#. (%%%###%#*  (%%%###%#/                                   // //////                 
       /%%%#*  .##. (%#    /##. (%(     #%#                                 *//////////                
      .#%(          (%(    ,##, (%/     #%%(         @@                    ****((((((((/               
       #%#          (%%#((#%%(  (%%#(/(#%%%/        %@%@                  *((((((((((((((              
       *#%%#*  .##. (%%####(.   (%%%%%%%%%#                              ((((((     ((((((             
         (#%%##%%#. (%#         (%%%%%%%#*                              (((((((      ((((((            
                                                                       (((((((*      ((((((            
                                                                      ((((((,          ((((((*         
                                                                     ((/                    ((/        
                                                                                               *       
The complete guide to installing Arch and configuring Arch on the GPD Pocket 2, based on all available online resources I can find on the GPD Pocket 2.

It is also relevant to other distros in the configuration parts.

I will do my best to explain everything ;)

## Installation
Download the live boot ISO from Arch Linux and write it to a USB stick.

Boot into the USB by going into the Bios, spam `Del` or hold `Fn + F12`.

~~Rememeber to boot into the `UEFI` partition.~~ (Not too important).

### Rotating the console
Let's start by rotating the console.
```
echo 1 > /sys/class/graphics/fbcon/rotate_all
```

### Internet
Connect via WiFi or plug in your an Ethernet Adaptor.
#### WiFi
Select your network and use dynamic IP
```
wifi-menu
systemctl restart dhcpcd.service
```
#### Checking connection
Wait a good 5-10 seconds, then check your connection by pinging a website.
```
ping archlinux.org
```
### Check your boot mode
You should be in `UEFI`, if not, you will need to boot into the `UEFI` partition on your USB stick.
```
ls /sys/firmware/efi/efivars
```
### Update your clock
```
timedatectl set-ntp true
```
### Partitioning
Check the storage devices you have and partitions on your Pocket 2 by:
```
lsblk
```
You should have a storage device `/dev/mmcblk0`, which is your main drive on the GPD Pocket 2.
#### GUID Partition Table (GPT)
__This is the part where you back up your files if you haven't already!__

We will be repartitioning the drive, __this will delete everything__.

Fire up `gdisk` and select the main drive `/dev/mmcblk0`:
```
gdisk /dev/mmcblk0
```
Use `m` to show available options.

__Keep in mind there will be no changes to the disk until you write them.__

To create a new empty GUID partition table, press `o` followed by `y`.
#### Boot Partition
To create a new partition, press `n` followed by `Enter`.

This will be the first partition, press `1` followed by `Enter`.

We want our partition to start at the default first sector, so just `Enter`.

I usually use 256MB for my boot partition, but you can choose 512MB just to be safe.

`+256M` follow by `Enter`.

This will be an EFI partition type, so `ef00` followed by `Enter`.
#### Swap Partition (Optional)
Swap partition is optional, but I would like the ability to hibernate.

To create a new partition, press `n` followed by `Enter`.

This will be the second partition, press `2` followed by `Enter`.

We want our partition to start at the default first sector, so just `Enter`.

I usually use my RAM size + 2GB for my swap partition, but you can also just have 8GB for the Pocket 2.

`+10G` follow by `Enter`.

This will be a linux swap partition, so `8200` followed by `Enter`.
#### Root Partition
I usually just have a root partition, if you want a home and a root partition see
[Arch Wiki](https://wiki.archlinux.org/index.php/installation_guide).

To create a new partition, press `n` followed by `Enter`.

This will be the third partition, press `3` followed by `Enter`.

We want our partition to start at the default first sector, so just `Enter`.

I will use the rest of the disk for Arch, so just `Enter`.

This will be a x86_64 root partition, so `8304` followed by `Enter`.
#### Write changes
Use `p` to see check your layout, start over if you mess something up.

Use `w` to write changes to your disk, and `y` to confirm.
### Formatting Partitions and Using Swap
Check your partition names:
```
lsblk
```
For the boot partition we will use fat32.
```
mkfs.fat -F32 /dev/mmcblk0p1
```
Now make the swap partition.
```
mkswap /dev/mmcblk0p2
```
For the root partition we will use ext4.
```
mkfs.ext4 /dev/mmcblk0p3
```
### Mounting the partitions
__I will not be encrypting the drives!__

Mount our root partition:
```
mount /dev/mmcblk0p3 /mnt
```
Create a folder for out boot partition:
```
mkdir /mnt/boot
```
Mount the boot partition:
```
mount /dev/mmcblk0p1 /mnt/boot
```
Activate the swap partition:
```
swapon /dev/mmcblk0p2
```
### Installing Arch Linux
To install Arch Linux:
```
pacstrap /mnt base base-devel dialog wpa_supplicant intel-ucode vim
```
#### Why do I need all these packages?
`base`: this is your system, kernel, etc.

`base-devel`: for other system essential stuff such as `sudo`, `pacman`, `systemd`.

`dialog`: __you will need this to connect to WiFi upon reboot into your system__ (I learned it the hard way FYI).

`wpa_supplicant`:__you will need this for WPA networks__.

`intel-ucode`: this is the microcode for intel based CPUs.

`vim`: optional, but makes your life easier later ~~(or harder)~~.


