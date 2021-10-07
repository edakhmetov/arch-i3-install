# Step by step installation of Arch Linux with i3 window manager

## load keymaps
``` 
localectl list-keymaps | grep ru
loadkeys ru
```

## IP setup
```
ip a
```
##### if connected with ethernet, ip will be available automatically, to connect wifi do following
```
iwctl
```
##### shows list of wireless adapters
```
device list
```
##### use the name from the device list
```
station [device-name] scan
```
##### show list of available networks
```
station [device-name] get-networks
```
##### use device name and network name from previous steps
```
station [device-name] connect [network-name]
```
##### you will be prompted to enter your wi-fi password
```
exit
```
##### type again to confirm that you have ip
```
ip a
```

## Synchronize the network time protocols
```
timedatectl set-ntp true
```

## Use reflector to setup mirrorlist
```
reflector -c Canada -a 6 --sort rate --save /etc/pacman.d/mirrorlist
```

