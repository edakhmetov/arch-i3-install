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
Using gdisk, create uefi partition, swap partition, and root partition
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

# creating root partition (accept all defaults if this is the last partition)

[default]
[default]
[default]	#takes all of the remaining space
[default]	#8300 for linux filesystem

w 		#write changes to the disk

lsblk		#confirm that you created proper partitions
```
# Format partitions
```
mkfs.vfat /dev/sda1		#format UEFI partition
mkswap /dev/sda2		#format swap partition
swapon /dev/sda2		#activate swap
mkfs.ext4 /dev/sda3		#format root partition
```
# Mount Partitions
```
mount /dev/sda3 /mnt		#mount root directory
mkdir -p /mnt/boot/efi		#create boot directory
mount /dev/sda1 /mnt/boot/efi	#mount boot directory
```
# Install base system
```
pacstrap /mnt base linux linux-firmware git nano intel-ucode		#or type amd-ucode if you have amd processor
```
# Generate filesystem table
```
genfstab -U /mnt >> /mnt/etc/fstab
```
# Now the installation
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
pacman -S grub efibootmgr networkmanager network-manager-applet dialog wpa_supplicant mtools dosfstools reflector base-devel linux-headers avahi xdg-user-dirs xdg-utils gvfs gvfs-smb nfs-utils inetutils dnsutils bluez bluez-utils cups hplip alsa-utils pulseaudio pulseaudio-bluetooth bash-completion openssh rsync  acpi acpi_call tlp virt-manager qemu qemu-arch-extra edk2-ovmf bridge-utils dnsmasq vde2 openbsd-netcat iptables-nft ipset firewalld flatpak sof-firmware nss-mdns acpid os-prober ntfs-3g terminus-font
pacman -S xf86-video-amdgpu
#if nvidia, pacman -S install nvidia nvidia-utils nvidia-settings
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB
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
useradd -mG wheel edd	#change edd to the preferred username
passwd edd 		#enter the password for the user
EDITOR=nano visudo	#find and uncomment %wheel ALL=(ALL) ALL, save and exit
exit
umount -a
reboot			#change in bios to load from the drive, not iso
```
