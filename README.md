# Step by step installation of Arch Linux with i3 window manager

# load keymaps
``` 
localectl list-keymaps | grep ru
loadkeys ru
```

# IP setup
```
ip a # if connected with ethernet, ip will be available automatically
```
# Connect Wi-Fi
```
iwctl

device list	#shows list of wireless adapters

station [device-name] scan	#use the name from the device list

station [device-name] get-networks	#show list of available networks

station [device-name] connect [network-name]	#enter wi-fi password when prompted

exit

ip a	#type again to confirm that you have ip
```
# Synchronize the network time protocols
```
timedatectl set-ntp true
```
# Use reflector to setup mirrorlist then synchronize them
```
reflector -c Canada -a 6 --sort rate --save /etc/pacman.d/mirrorlist

pacman -Syy
```
# Disk partitioning
Using gdisk, create uefi partition, swap partition, root partition, and home partition
```
lsblk	#get the disk name, which will be used in next steps

gdisk /dev/sda	#sda is the name of the disk from previous step

n	#after each partition, type n to create new partition

# uefi partition

[default]		#partition number
[default]		#first sector (default is ok)
+260M			#last sector (260mb for UEFI)
ef00			#partition type code (ef00 for UEFI)

# swap partition

[default]
[default]
+4G		#4gb swap
8200		#8200 for linux swap

# creating root partition

[default]
[default]
+40G		#40Gb, change if you want more or less
[default]	#8300 for linux filesystem

# creating home partition

[default]
[default]
[default]	#takes all of the remaining space
[default]

w 		#write changes to the disk

lsblk		#confirm that you created proper partitions
```
# Format partitions
```
mkfs.vfat /dev/sda1		#format UEFI partition

mkswap /dev/sda2		#format swap partition

swapon /dev/sda2		#activate swap

mkfs.ext4 /dev/sda3		#format root partition

mkfs.ext4 /dev/sda4		#format home partition
```
# Mount Partitions
```
mount /dev/sda3 /mnt		#mount root directory

mkdir -p /mnt/boot/efi		#create boot directory

mount /dev/sda1 /mnt/boot/efi	#mount boot directory

mkdir /mnt/home			#create home directory
	
mount /dev/sda4 /mnt/home	#mount home partition
```
# Install base system
```
pacstrap /mnt base linux linux-firmware git nano intel-ucode		#or type amd-ucode if you have amd processor
```
# Generate filesystem table
```
genfstab -U /mnt >> /mnt/etc/fstab
```
# Installation of system packages and system setup
```
arch-chroot /mnt

timedatectl list-timezones | grep Toronto	#find the closest timezone

ln -sf /usr/share/zoneinfo/America/Toronto /etc/localtime	#save local timezone

hwclock --systohc		#synchronize hardware clock with system

nano /etc/locale.gen		#find and uncomment en_US.UTF-8 UTF-8

locale-gen			#generate locale

nano /etc/locale.conf		#type in LANG=en_US.UTF-8, then save and close

nano /etc/vconsole.conf		#add KEYMAP=ru (do it if you added keymap in the beginning)

nano /etc/hostname		#type in any name for host machine

nano /etc/hosts			

#add following lines
127.0.0.1	localhost
::1		localhost
127.0.1.1	hostname.localdomain	hostname	#replace hostname with the name you used
#save and exit

passwd		#enter the password for root user

# remove the tlp package if installing on a desktop or vm
pacman -S grub efibootmgr networkmanager network-manager-applet dialog wpa_supplicant mtools dosfstools reflector base-devel linux-headers avahi xdg-user-dirs xdg-utils gvfs gvfs-smb nfs-utils inetutils dnsutils bluez bluez-utils cups hplip alsa-utils pulseaudio pulseaudio-alsa pulseaudio-bluetooth pulseaudio-jack bash-completion openssh rsync acpi acpi_call tlp virt-manager qemu qemu-arch-extra edk2-ovmf bridge-utils dnsmasq vde2 openbsd-netcat iptables-nft ipset firewalld flatpak sof-firmware nss-mdns acpid os-prober ntfs-3g terminus-font playerctl scrot zip unzip unrar rofi vlc

pacman -S xf86-video-amdgpu		#if nvidia, pacman -S install nvidia nvidia-utils nvidia-settings

grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB

#grub-mkconfig -o /boot/grub/grub.cfg

nano /etc/default/grub		#update GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet video=1920x1080", save and exit

grub-mkconfig -o /boot/grub/grub.cfg

systemctl enable NetworkManager

systemctl enable bluetooth

systemctl enable cups.service

systemctl enable sshd

systemctl enable avahi-daemon

systemctl enable tlp 	#comment this command out if you didn't install tlp, see above

systemctl enable reflector.timer

systemctl enable fstrim.timer

systemctl enable libvirtd

systemctl enable firewalld

systemctl enable acpid

#useradd -mG wheel edd	#old way, change edd to the preferred username

useradd -m edd

passwd edd		#enter the password for the user

usermod -aG libvirt edd

#EDITOR=nano visudo	#old way, find and uncomment %wheel ALL=(ALL) ALL, save and exit

echo "edd ALL=(ALL) ALL" >> /etc/sudoers.d/edd

nano /etc/mkinitcpio.conf	#add greaphic cards to MODULES (i915 for intel, amdgpu for amd, nvidia), can add multiple

mkinitcpio -p linux

exit

umount -R /mnt

reboot			#change in bios to load from the drive, not iso
```
# Installation of packages and i3 wm
Once you boot up, enter your username and password. Use `sudo nmtui` to connect to wi-fi
```
timedatectl set-ntp true

sudo hwclock --systohc

sudo reflector -c Canada -a 6 --sort rate --save /etc/pacman.d/mirrorlist

sudo pacman -Syy

sudo firewall-cmd --add-port=1025-65535/tcp --permanent

sudo firewall-cmd --add-port=1025-65535/udp --permanent

sudo firewall-cmd --reload

# add tap to click and natural scrolling for touchpad
sudo mkdir -p /etc/X11/xorg.conf.d

sudo nano /etc/X11/xorg.conf.d/90-touchpad.conf

# enter the following
Section "InputClass"
	Identifier "touchpad"
	MatchIsTouchpad "on"
	Driver "libinput"
	Option "Tapping" "on"
	Option "NaturalScrolling" "on"
EndSection
# save and exit

sudo pacman -S xf86-video-intel xf86-video-amdgpu xorg i3-gaps i3blocks i3lock i3status dmenu lightdm lightdm-gtk-greeter lightdm-gtk-greeter-settings firefox nitrogen picom lxappearance thunar papirus-icon-theme xfce4-terminal gimp neofetch htop dunst

sudo pacman -S dina-font tamsyn-font bdf-unifont ttf-bitstream-vera ttf-croscore ttf-dejavu ttf-droid gnu-free-fonts ttf-ibm-plex ttf-liberation ttf-linux-libertine noto-fonts ttf-roboto tex-gyre-fonts ttf-ubuntu-font-family ttf-anonymous-pro ttf-cascadia-code ttf-fantasque-sans-mono ttf-fira-mono ttf-hack ttf-fira-code ttf-inconsolata ttf-jetbrains-mono ttf-monofur adobe-source-code-pro-fonts cantarell-fonts inter-font ttf-opensans gentium-plus-font ttf-junicode adobe-source-han-sans-otc-fonts adobe-source-han-serif-otc-fonts noto-fonts-cjk noto-fonts-emoji

# install yay aur helper
git clone https://aur.archlinux.org/yay.git
cd yay/
makepkg -si

yay -S jq ttf-weather-icons masterpdfeditor-free polybar ttf-iosevka-term ttf-iosevka polybar otf-nerd-fonts-fira-code siji-git

sudo systemctl enable lightdm

reboot
```
# This completes the Arch Linux installation with i3 window manager
