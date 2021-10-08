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
## Using gdisk, create uefi partition, swap partition, and root partition
```
lsblk	#get the disk name, which will be used in next steps

gdisk /dev/sda	#sda is the name of the disk from previous step

n	#after each partition, type n to create new partition

# uefi partition

[default]		#partition number
[default]		#first sector (default is ok)
+260M		#last sector (260mb for UEFI)
ef00		#filesystem type code (ef00 for UEFI)

# swap partition

[default]
[default]
+4G		#4gb swap
8200		#8200 for swap

# creating root partition (accept all defaults if this is the last partition)

[default]
[default]
[default]
[default]

w 	#write changes to the disk
```
