# Arch Linux Installation Notes

### Pre-installation

1. Check if network works

   ```shell
   pint archlinux.org
   ```

   1. If you want to connect internet with wireless card, then just type

      ```shell
      lspci -k # find you wireless card, and we assume your wireless device is wlan0
      ip link set wlan0 up # turn up the wireless card

      # if your wireless card driver is ready, then use following command to scan 
      # a wifi signal. There are three import imformation you have to notice:
      # 1. SSID
      # 2. Signal
      # 3. Capabipity: ESS Prvacy ShortSlotTime (0x0411), then the signal is protected
      #      if RSN block exists: WPA2
      #      if WPA block exists: WPA
      #      if if no RSN and WPA block but have privacy, then the WEP 
      iw dev wlan0 scan | less

      # if NO encryption
      iw dev wlan0 connect "SSID"
      # if WEP 
      iw dev wlan0 connect "SSID" key 0:your_key
      # if WPA/WPA2
      wpa_supplicant -B -i wlan0 -c <(wpa_passphrase SSID your_key)

      # after connect successfully, get your ip by DHCP
      dhcpcd wlan0
      ```

   2. Or, another easier way: 

      ```shell
      sudo wifi-menu
      ```

2. Update system clock

   ```shell
   timedatectl set-ntp true 				# ensure clock is accurate
   timedatectl list-timezones 				# list timezones
   timedatectl set-timezone Asia/Taipei 	# set timezone to your location
   timedatectl status						# check if setting is correct
   ```

3. Partition your disk

   ```shell
   # list all disk, assume your disk is sdx
   fdisk -l

   # umount your disk if it is mounted
   umount /dev/sdx

   # use fdisk to partition your disk
   fdisk /dev/sdx

   # format your disk after partitioned
   # assume n is the partition number you want to format
   mkfs.ext4 /dev/sdxn

   # if you want to create a EFI partition
   mkfs.fat -F32 /dev/sdxn

   # if you want to create a swap partition
   mkswap /dev/sdxn
   ```

4. mount your disk

   ```shell
   # mount the root partition to /mnt
   mount /dev/sdxn /mnt

   # mount the remaining partition into mnt
   mkdir /mnt/boot
   mount /dev/sdxn /mnt/boot
   ```

### Installation

1. install the base packages

   ```shell
   pacstrap /mnt base
   ```

2. configure the system

   1. fstab: generate a fstab file

      `genfstab -U /mnt >> /mnt/etc/fstab`

   2. change root into the new system

      `arch-chroot mnt`

   3. set timezone

      `ln -sf /usr/share/zoneinfo/Asia/Taipei /etc/localtime`

      replace Asia and Taipei to where your region and city

   4. run hsclock to generate `/etc/adjtime`

      `hwclock --systohc`

   5. uncomment `en_US.UTF-8 UTF-8` in /etc/locale.gen, and run `locale-gen`

   6. add `LANG=en_US.UTF-8` into /etc/locale.conf

   7. add a file `/etc/hostname`, and the content is the name of your host

   8. change  `/etc/hosts` and add following content

      ```shell
      127.0.0.1	localhost.localdomain	localhost
      ::1			localhost.localdomain	localhost
      127.0.1.1	myhostname.localdomain	myhostname
      ```

   9. setup network -- I use netctl

      1. install netctl with pacman
      2. use `ip link` to check which interface you have
      3. copy the corresponding config file from `/etc/netctl/examples` to `/etc/netctl/`
      4. rename the copied file to name of the interface you use
      5. change the Interface in the config file to your interface name
      6. netctl enable \${name of your interface}
      7. netctl start \${name of your interface}

   10. setup swap space if you want

     1. get the UUID by `blkid /dev/sdxn`

     2. add it to `/etc/fstab`

        `UUID=xxx none swap sw 0 0`

   11. set root password by `passwd`

   12. setup bootloader (in vmware, it use UEFI)

       1. install grub and efibootmgr 

       2. creage grub boot menu

          ```shell
          grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=grub
          ```

       3. setup grub

          ```shell
          grub-mkconfig -o /boot/grub/grub.cfg
          ```

3. exit arch-chroot and reboot

## Following theme is optional

### yaourt 

yaourt can let you install pacakge from AUR in a very easy way. Just replace pacman with yaourt, all other option is the same.

Note: You must use an non-root account and the account must have sudo permission.

```shell
pacman -S --needed base-devel git wget yajl
git clone https://aur.archlinux.org/package-query.git
git clone https://aur.archlinux.org/yaourt.git
cd package-query/ && makepkg -si && cd ..
cd yaourt/ && makepkg -si && cd ..
```

### Chinese fonts

```shell
pacman -S adobe-source-han-serif-tw-fonts adobe-source-han-sans-tw-fonts
```

### GUI installation

1. install xorg, xorg-xinit

2. select your favorite desktop environment and install it via pacman. 
   For example, plasma(`pacman -S plasma`)

3. configure your .xinitrc file under your home directory

   ```shell
   exec plasma
   ```

4. if want to start GUI when login, add following into your shell profile

   ```shell
   if [[ ! $DISPLAY && $XDG_VTNR -eq 1 ]]; then
   	exec startx
   fi
   ```

5. start GUI by `startx`

### GUI network manager

To let network device work like windows or macOS(connect to known network automatically), we have to install `networkmanager`. After installation, enable the NetworkManager.service.

```shell
sudo systemctl enable NetworkManager.service
sudo systemctl start NetworkManager.service
```

### Graphic Driver

```shell
# for intel integrated graphic card
pacman -S mesa 

# for nvidia graphic card. 
# You have to check if your graphic card support this latest nvidia driver first.
pacman -S nvidia 
```

Unsupported devices and its driver: [here](https://wiki.archlinux.org/index.php/NVIDIA#Unsupported_drivers).

If your computer have more than 1 graphic device(for example, intel integrated graphic card and nvidia grapnic card), then you have to install `bumblebee` to support *hybrid graphics* implementation. Bumblebee is NVIDIA optimus for linux roughly.

### Chinese input method

```shell
pacman -S ibus 
pacman -S ibus-chewing 		 # 注音䠼入法
pacman -S ibus-table-chinese # 倉頡、速成
```

## Appendix

### Frequently used pacman options

1. -Syu: update pacman database and software
2. -Sy: only update pacman database
3. -Ss: search package in database. Must run -Sy first for updating database.
4. -R: remove package
5. -Rs: remove package and all its dependencies

