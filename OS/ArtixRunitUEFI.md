### Increasing the Font Size
setfont LatGrkCyr-12x22

### Internet connection
ip a // Checking your internet setup and it will show your wifi-adapter
### Ethernet will work straight out of the box

### For wifi
ip link set wlan0 up // wlan0 or whatever wifi-adapter you have

### Incase of RF-kill error, it means wifi is blocked. then run the two commands below else ignore 
rfkill unblock wifi
ip link set wlan0 up // This time no error will be raised 

### Now Connecting to Wifi
connmanctl 
-> scan wifi
-> services // You will find your wifi name and wifi_(some random text)
-> agent on
-> connect wifi_ // then tab and few characters of your wifi_(text) and it will auto-complete
 //then it will ask for passphrase and you can connect then quit from connmanctl
-> quit

### Disk Partitioning
lsblk
cfdisk // open an interface to partitioning the disk
// compulsory partitioning efi root maybe even swap or if you want you can have home partition too 
/*
	My Preference
		efi = 520M
		swap = 2G
		root = 30G
		home = Max you want to give
*/

### Formating the partion
mkfs.fat -F32 $(efi disk name; eg /dev/sda1)
mkswap $(swap disk name; eg /sda2)
mkfs.ext4 $(root disk name; eg /dev/sda3)
mkfs.ext4 $(home disk name. if you want; eg /dev/sda4)

### Mounting the disk
swapon $(swap; eg /dev/sda2) // Starting swap
mount $(root; eg /dev/sda3) /mnt
mkdir /mnt/home
mkdir /mnt/boot
mount $(home; eg /dev/sda4) /mnt/home
mount $(boot; eg /dev/sda1) /mnt/boot

### Installing the  Artix system
basestrap /mnt base base-devel linux linux-headers linux-firmware runit elogind-runit vim nano intel-ucode 
// Depending your processor add amd-ucode or intel-ucode

### Setting up fstab 
fstabgen  -U /mnt >> /mnt/etc/fstab
// Your system checks the fstab while booting to mount your disks

### Chrooting into you new Artix system
// Basically getting into your Artix linux terminal
artools-chroot /mnt 

### Setting up Time Zone
timedatectl list-timezones | grep $(Your Place or region) // if u don't get a output it's fine
ln -sf /usr/share/zoneinfo/  //just click 'tab' twice and you will find country or continen then 'tab' twice you will fine the region

eg: ln -sf /usr/share/zoneinfo/Asia/Kolkata /etc/localtime

hwclock --systohc //sysnc with hardware clock

### Setting up Locale
nano /etc/locale.gen
/* 
	eg remove the hash on the language you want for english 
	remove the hash of "#en_US.UTF-8 UTF-8"
	then save and exit
*/

locale-gen

nano /etc/locale.conf
/*
	then type "LANG=en_US.UTF-8" if english or enter the language you used in locale
	then save and exit
*/ 

### Setting up the Keymap
nano /etc/vconsole.conf
/*
	Then type "KEYMAP=us" or enter the keymap that you choose in the beginning
*/

### Setting up hostname
nano /etc/hostname
/*
	Enter the name you wanna give the PC or Laptop
*/

### Setting up the hosts
nano /etc/hosts
/*
	Append this Data Below

	127.0.0.1	localhost
	::1		localhost
	127.0.1.1	$(hostname).localdomain $(hostname)

	then save and exit
*/

### Setting a Password for Root User
passwd //then it was ask you to enter a root password

### Adding other required packages
pacman -S grub efibootmgr networkmanager networkmanager-runit networkmanager-applet dialog linux-headers bluez bluez-runit bluez-utils alsa-utils pulseaudio pulseaudio-bluetooth xf86-video-intel xorg --ignore xorg-server-xdmx git xdg-utils xdg-user-dirs
/*
	xf86-video-amd for AMD processors instead of xf86-video-intel

	Incase u are dual booting with other distro or windows
	add a package called "os-prober"
*/

### Setting up Grub
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=Artix --removable --recheck 
/*
	Incase of Dual booting run the command 'os-prober' 
	Before running the 'grub-mkconfig'
*/
grub-mkconfig -o /boot/grub/grub.cfg

mkinitcpio -p linux

###  Enabling Startup Applications
ln -s /etc/runit/sv/NetworkManager /etc/runit/runsvdir/current

### Creating New User
useradd -mG wheel cherry // Instead of 'cherry' uses whatever username you want
passwd cherry //giving password to your new user

### Giving sudo privilages to your new user
EDITOR=nano visudo
/*
	Unhash this text "# %wheel ALL=(ALL) ALL"
	You will find it towards the end of the file
	Then save and exit
*/ 

### Now exit chroot
exit

### Now Unmount everything and reboot   
umount -a
reboot

### Enabling Startup Applications
ln -s /etc/runit/sv/bluetoothd /run/runit/service/bluetoothd
