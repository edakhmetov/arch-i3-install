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

device list # shows list of wireless adapters

station [device-name] scan # use the name from the device list

station [device-name] get-networks # show list of available networks

station [device-name] connect [network-name] # Prompted to enter wi-fi password

exit

ip a # type again to confirm that you have ip
```

## Synchronize the network time protocols
```
timedatectl set-ntp true
```

## Use reflector to setup mirrorlist then synchronize them
```
reflector -c Canada -a 6 --sort rate --save /etc/pacman.d/mirrorlist
pacman -Syy
```

## Disk partitioning
##### get the disk name, which will be used in next steps
```
lsblk 
```
##### using gdisk, create uefi partition, swap partition, and root partition
```
gdisk /dev/sda
```
##### creating uefi partition
```
n
[empty]
[empty]
+260M
ef00
```
##### creating swap partition
```
n
[empty]
[empty]
+4G
8200
```
##### creating root partition
```
n
[empty]
[empty]
[empty]
[empty]
```
##### write changes to the disk
```
w
```
