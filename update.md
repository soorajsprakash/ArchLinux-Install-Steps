* `loadkeys us`
* `rfkill unblock all`
* `iwctl`
    * `station wlan0 connect <SSID>`
    * `exit`

* `timedatectl set-ntp true`

* `lsblk`
* `cfdisk` or `fdisk` or `gdisk /dev/sda`
* 
* `mkfs.fat -F32...........


* `mount /dev/sda2 /mnt`
* `btrfs sv create @`
* `btrfs sv create home`

* `umount /mnt`

* `mount -o noatime,space_cache=v2,compress=zstd,ssd,discard=async,subvol=@ /dev/sda2 /mnt` 
* `mkdir /mnt/{boot,home}`
* `mount -o noatime,space_cache=v2,compress=zstd,ssd,discard=async,subvol=@ /dev/sda2 /mnt/home` 
* `mount /dev/sda1 /mnt/boot`



* `pacstrap /mnt base base-devel linux linux-firmware linux-headers git nano ntfs-3g man intel-ucode`
* `genfstab -U /mnt >> /mnt/etc/fstab`
* `arch-chroot /mnt`

* `nano /etc/mkinitcpio.conf`
    * add `btrfs` to the modules section
* `mkinitcpio -p linux`

* `chmod +x base.sh`


**systemd-boot**
`bootctl --path=/boot install`

edit loader.conf

`nano /boot/loader/loader.conf`

```
timeout 2
default arch
```

`nano /boot/loader/entries/arch.conf`

```
title   Arch Linnux
linux   /vmlinuz-linux
initrd /initramfs-linux.img
options rootflags=subvol=@ rw video=1920x1080
```

* paru
```
git clone https://aur.archlinux.org/paru-bin
makepkg -si
```

```
paru timeshift-bin timeshift-autosnap zramd

sudo systemctl enable --now zram.service
systemctl enable fstrim.service
...acpid? (for what?)

nano /etc/default/zramd
```


`chmod +x <filename>`


* check for timeshit backups to show in systemd-boot menu
* create bash scripts for auto install faster