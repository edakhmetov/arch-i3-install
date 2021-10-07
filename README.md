Step by step installation of Arch Linux with i3 window manager

# load keymaps 
localectl list-keymaps | grep ru
loadkeys ru

# check for ip
ip a
# if connected with ethernet, ip will available automatically
# following is for the wifi
iwctl
# show list of wireless adapters
device list
# use the name from the device list
station [device-name] scan
# show list of available networks
station [device-name] get-networks
# use device name and network name from previous steps
station [device-name] connect [network-name]
# you will be prompted to enter your wi-fi password
exit
# type again to confirm that you have ip
ip a
