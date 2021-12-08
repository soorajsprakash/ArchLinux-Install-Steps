# THIS IS NOT AN INSTALLATION SCRIPT, BUT THE STEPS I FOLLOWED TO INSTALL ARCH LINUX
#### Source: [ArchWiki](https://wiki.archlinux.org/title/Installation_guide#Pre-installation) (90%) --> will update source for each topic later


| Stuff | My Stuff |
| --- |---:|
| DE         | GNOME |
| GPU        | INTEL+NVIDIA |
| SWAP       | SWAPFILE |
| DRIVE      | SSD |
| SHELL      | ZSH |
| KERNEL     | LINUX & LINUX-LTS |
| EDITOR     | NANO |
| NETWORK    | WIFI |
| FILESYSTEM | BTRFS |
| BOOTLOADER | GRUB |

---------------------------------------------


## Update the system clock
1. `timedatectl set-ntp true`


## Set the console keyboard layout
(default is US, so you don't have to do this)

2a. `localectl list-keymaps | list <your-keyboard-layout>`
> Eg: `localectle list-keymaps | list us`

2b. `loadkeys "<your-keymap>"`
> Eg: `loadkeys "us"`

## Connect to the internet (wifi)

3. `iwctl` This opens an interactive prompt to connect to wifi


  3a. `device list` This lists all the wifi adapter connected
  
  
  3b. `station <device-name> scan`
  > `station wlan0 scan` (assuming wlan0 is your adapter) 


  3c. `station wlan0 get-networks` This returns the scanned neworks near you.


  3d. `station wlan0 connect <SSID>`
  > Eg: `station wlan0 connect FTTH-DDC` Then enter passphrase.


4. Wait for few seconds & exit out of there by typing `exit`.


## Partition the disks 


### Creating physical partition: 


5. `lsblk` this lists out the details of the block device connected.

6. Use `cfdisk` or `fdisk` or something else to make your partitions.

> Create a partition with more than 300MB for boot.
> 
> Create root partition
> 
> A Home partition if needed.
> 
> Swap partition if necessary, else we could use swapfile
> 
> Also create any other partitions that you need. We can use these as Local Disk D,E,F as in windows. 

7. Format the partitions with a file system.

> I made a boot partition `/dev/sda1` of 512MB size (your partition name/number may differ). 
> 
> Made a root partition `/dev/sda2` of 350GB.
> 
> Made another partition `/dev/sda3` as a seperate partition for my Projects (130Gb) .
> 
> Have decided not to make a seperate HOME partition as I've got only a 480GB ssd.
> Eventually HOME or ROOT may want more space, so by not making a seperate HOME, i got to not physically limit HOME or ROOT,
>  & using a seperate partition for all my Important project files.

7a. `mkfs.fat -F32 /dev/sda1` Formatting boot as EFI:

7b. `mkfs.btrfs /dev/sda2` Formatting root as btrfs (butter filesystem):

7c. `mkfs.btrfs -L Projects /dev/sda3` Formatting my other partition also as btrfs, also labelling the partition as "Projects":



### Creating btrfs subvolumes:

8. `mount /dev/sda2 /mnt` Mounting root partition into "/mnt"

9. `btrfs subvolume create /mnt/@` Make a root subvolume inside our root partition
* if you want to use timeshft, you need to make root subvolume as `@root` instead of `@` which is for using with snapper. 
* I don't use both, so it doesn't matter for me. 

10. `btrfs su cr /mnt/@home` Same command, but shortened version, create a home subvolume (because we can), & this ain't a partition with physical size defined. so...

11. `btrfs su cr /mnt/@swap` We have to create a swap subvolume in btrfs to use a swapfile

12. `umount /mnt` Unmount the `/mnt` partition that we mounted.



### Mounting the subvolumes:

13. `mkdir -p /mnt/{boot,home,.swap}` This makes boot, home & swap directory inside the /mnt.

14. `mount -o noatime,ssd,compress=lzo,space_cache=v2,subvol=@ /dev/sda2 /mnt` Mounts root subvolume into root folder.

15. `mount -o noatime,ssd,compress=lzo,space_cache=v2,subvol=@home /dev/sda2 /mnt/home` Mounts home subvolume into home folder.

18. `mount -o nodatacow,subvol=@swap /dev/sda2 /mnt/swap` Mounts swap subvolume into swap folder.

19. `mount /dev/sda1 /mnt/boot` Mounts the boot partition into the boot folder.


## Installing the Base System

20. `pacstrap /mnt base base-devel linux linux-headers linux-lts linux-lts-headers linux-firmware nano intel-ucode networkmanager btrfs-progs wpa_supplicant grub efibootmgr dosfstools grub-btrfs` Installs the base package needed + few extra for me.
21. `genfstab -U /mnt >> /mnt/etc/fstab` This generates a fstab file to go.
22. `arch-chroot /mnt` Change root to the base system install

## Installation of yay binary

- `git clone https://aur.archlinux.org/yay-bin.git ~/yay-bin`
- `cd ~/yay-bin`
- `makepkg -si`
- `cd ~`
- `rm -rf ~/yay-bin`

## GPU Drivers (Intel + Nvidia Hybrid)

- `yay -S mesa vulkan-intel intel-media-driver nvidia-dkms libva-vdpau-driver-vp9-git optimus-manager optimus-manager-qt`


## Create a swap file

23. `truncate -s 0 /.swap/swapfile`
24. `chattr +C /.swap/swapfile`
25. `btrfs property set /.swap/swapfile compression none`
26. `dd if=/dev/zero of=/.swap/swapfile bs=1G count=5 status=progress` This creates a swapfile of 5 Gb.
27. `chmod 600 /.swap/swapfile` Sets the right permission
28. `mkswap /.swap/swapfile` Format the file as swap
29. `swapon /.swap/swapfile` Turn on the swap file
30. `nano /etc/fstab` Edit the fstab file to add an entry for the swap file: `/.swap/swapfile none swap defaults 0 0`


## [OPTIONAL] Add resume support for hibernation

31. We have to pass 2 params to the `GRUB_CMDLINE_LINUX_DEFAULT` param under `/etc/default/grub`. For that, do the following: 

31a. Find the UUID of our / from fstab by runnng `cat /etc/fstab`
* Append this UUID as "resume=UUID={your_uuid}" to the end of `GRUB_CMDLINE_LINUX_DEFAULT`, without the quotes and curly braces, but within the quotes of the `CMDLINE` parameter.

31b. To find resume_offset: 
* Download the file`curl -o btrfs_map_physical https://raw.githubusercontent.com/osandov/osandov-linux/master/scripts/btrfs_map_physical.c` 
* Compile it `gcc -O2 -o btrfs_map_physical btrfs_map_physical.c`
* Run the file generated after compilation, pointing to our swap file: `./btrfs_map_physical /.swap/swapfile`
* It will give output similar to this:
```
FILE OFFSET  EXTENT TYPE  LOGICAL SIZE  LOGICAL OFFSET  PHYSICAL SIZE  DEVID  PHYSICAL OFFSET
0            regular      4096          2927632384      268435456      1      4009762816
4096         prealloc     268431360     2927636480      268431360      1      4009766912
268435456    prealloc     268435456     3251634176      268435456      1      4333764608
536870912    prealloc     268435456     3520069632      268435456      1      4602200064
805306368    prealloc     268435456     3788505088      268435456      1      4870635520
1073741824   prealloc     268435456     4056940544      268435456      1      5139070976
1342177280   prealloc     268435456     4325376000      268435456      1      5407506432
1610612736   prealloc     268435456     4593811456      268435456      1      5675941888
```

* Take note of what value comes first under `PHYSICAL_OFFSET`, here in this example, it's **4009762816**.
* Also note the pagesize, which is the first value under `LOGICAL_SIZE`, here its **4096**.  (pagesize can also be found with `getconf PAGESIZE`) 
* Note down the `resume_offset` by dividing the PHYSICAL_OFFSET with the PAGESIZE.
* Append this to the end of `GRUB_CMDLINE_LINUX_DEFAULT` as "resume_offset={your_offset}", without the quotes and curly braces, but within the quotes of the `CMDLINE` parameter.

32. Add "reume" option (without the quotes) to the end of the HOOKS section in `/etc/mkinitcpio.conf`, within the quotes of HOOKS.
> We need to edit this file again, so we will generate the initramfs later.



## More System Configuration


33. Set timezone `ln -sf /usr/share/zoneinfo/Asia/Kolkata /etc/localtime`
34. Set hw clock to system clock `hwclock --systohc`
35. Uncomment `en_IN UTF-8` and `en_US.UTF-8 UTF-8` from `/etc/locale.gen` file, to set locale
36. Run `locale-gen` to generate locale file
37. Edit `/etc/locale.cong` and add `"LANG=en_US.UTF-8"`
38. Add the keymap to `/etc/vconsole.conf` For ex: `KEYMAP=us` or `KEYMAP=in`
39. Set a hostname for your PC by adding the name to `/etc/hostname` file. For as example, I set it to archlinux
40. Edit hosts file  `/etc/hosts` and add the following for networking part
``` 
127.0.0.1   localhost
127.0.1.1   archlinux
::1        archlinux.localdomain    archlinux
```
> (It's a TAB & not space)
41. Create your root password by typing `passwd`
42. Enable `multilib` repo to run 32 bit applications: Uncomment `[multilib]` and `Include` from `etc/pacman.conf`
43. Install other package that you need/might need
`pacman -S git reflector zsh zsh-completions bluez cups cups-pdf xdg-utils xdg-user-dirs xdg-user-dirs-gtk xdg-desktop-portal-gtk xdg-desktop-portal-gnome alsa-utils alsa-card-profiles pulseaudio pulseaudio pulseaudio-alsa pulseaudio-bluetooth pulseaudio-equalizer inetutils firefox vlc lollypop gimp shotwell transmission telegram-desktop steam wine winetricks mousetweaks`


## Installing Gnome DE and other drivers


43. Install intel drivers. `pacman -S mesa lib32-mesa mesa vulkan-intel intel-media-driver`
44. Install xorg group  `pacman -S xorg`
45. Install nvidia drivers `pacman -S nvidia nvidia-utils lib32-nvidia-utils nvidia-lts `
46. Install gnome: `pacman -S gnome gnome-extra gdm`


## Initramfs config


47. Edit `/etc/mkinitcpio.conf` & do the following:


47a. Add `"btrfs"` to the MODULES section.

47b. Add `"resume"` to the end of HOOKS section, just before "filesystems" This is for resume support for Hibernation.

47c. Add `"numlock"` to HOOKS section.

48. Generate initramfs:
```
mkinitcpio -p linux
mkinitcpio -p linux-lts
```



## Grub config, Add user, Enable services & Finishing up


49. Install GRUB: `grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB`
50. Generate Grub config: `grub-mkconfig -o /boot/grub/grub.cfg`
51. Create User (archguy for this example) & add it to groups:
```
useradd -mG wheel -s /bin/zsh archguy
passwd archguy
```
52. Type in `EDITOR=nano visudo` and uncomment the line `"wheel ALL=(ALL) ALL"`
53. Enable Services:
```
systemctl enable NetworkManager
systemctl enable bluetooth.service
systemctl enable cups.service
```
54. Exit from chroot by running `exit`
55. Unmount all `umount -a`
56. Reboot `reboot`
