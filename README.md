# single-amd-gpu-passthrough
simple, single amd gpu passthrough on intel platform (only)

# Prerequisite
- OS : Arch linux
- Knl: linux-zen
- DE : Plasma (wayland) 
- GPU: RNDA2  
- CPU: 12th-gen(K) Intel 
- VT-d and VT-x capable motherboard 

# Part -1 : Motherboard Bios
1) Enable VT-d and VT-x in your motherboards BIOS 
2) Disable REBAR and 4G decode 
3) Disable CSM

# Part -2 : Bootloader and IOMMU 
1) Add the following parameters to your bootloader: intel_iommu=on iommu=pt
2) Restart 
3) Follow the guide: https://gitlab.com/risingprismtv/single-gpu-passthrough/-/wikis/3)-IOMMU-Groups 
  
  Note down the Bus ID and Vendor ID for your GPU(dGPU) and your intergrated GPU (iGPU): 
 ```
   -VGA      (example) 03:00.0 and 1002:73ff  <- dGPU
 
   -AUDIO    (example) 03:00.1 and 1002:ab28  <- dGPU
   
   -VGA      (exmaple) 00:02.0 and 8086:4680  <- iGPU
```
# Part -3 : Libvirt and Qemu 
Follow the guide: https://gitlab.com/risingprismtv/single-gpu-passthrough/-/wikis/4)-Configuration-of-libvirt

# Part -4 : Virtual Machine (VM)
1) Follow the guide: https://gitlab.com/risingprismtv/single-gpu-passthrough/-/wikis/5)-Configuring-Virtual-Machine-Manager
2) (recommended) Name your VM: **win10** 

    This way, we have less scripts to edit in Part -9

# Part -5 : Inputs 
Pass mouse and keyboard to the VM

1) Follow the guide: https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF#Passing_keyboard/mouse_via_Evdev 
2) Navigate to /dev/input/by-id/ directory, here we want to find your the name of your  keyboard and mouse
3) Perform the test provided by the guide and note the device names 
4) Add the folowing to your VM xml file (replace MOUSE_NAME and KEYBOARD_NAME with the names you found) : 
                                     Example from previous link
 ```     
      <input type='evdev'>
      <source dev='/dev/input/by-id/MOUSE_NAME'/>
    </input>
    <input type='evdev'>
      <source dev='/dev/input/by-id/KEYBOARD_NAME' grab='all' repeat='on' grabToggle='ctrl-ctrl'/>
    </input>
```

# Part -6 : Audio 
*PLANNED*


# Part -7 : GPU VBios
Warning the following may result in permanent DAMAGE, follow at your own RISK!

1) Find your gpu vbios here and Download it: https://www.techpowerup.com/vgabios/
2) Rename the Vbios file to: patched.rom
3) Create a Directory to store and use the vbios:
 ```
          cd ~/Downloads  
          sudo mkdir /var/lib/libvirt/vgabios
          cp ./patched.rom /var/lib/libvirt/vgabios/
          cd /var/lib/libvirt/vgabios/
          sudo chmod -R 660 patched.rom
          sudo chown yourusername:yourusername patched.rom
   ```  
4) Adding the vbios to the VM
5) Follow the guide: https://gitlab.com/risingprismtv/single-gpu-passthrough/-/wikis/8)-Attaching-the-GPU-to-your-VM
6) Directory should be: var/lib/libvirt/vgabios/patched.rom
          
# Part -8 : Conversions 
1) We need to convert the Bus ID and Vender id to the following formart: 


  dGPU VGA 
  ``` 
      03:00.0   to 0000:03:00.0
      1002:73ff to 1002 73ff
  ```
  dGPU AUDIO
  ``` 
      03:00.1   to 0000:03:00.1
      1002:ab28 to 1002 ab28
  ```
  iGPU VGA
  ``` 
      00:02.0   to 0000:00:02.0
      8086:4680 to 8086 4680
  ```
  After conversion its should look like this: 
  ```
   -VGA      (example) 0000:03:00.0 and 1002 73ff  <- dGPU
 
   -AUDIO    (example) 0000:03:00.1 and 1002 ab28  <- dGPU
   
   -VGA      (exmaple) 0000:00:02.0 and 8086 4680  <- iGPU
```
  
  # Part -9 : Scripts and Startups 
  1) Follow the guide: https://gitlab.com/risingprismtv/single-gpu-passthrough/-/wikis/7)-scripts-&-logfiles
  2) After you have completed the installation 
  3) Edit */bin/vfio-startup.sh* to the following: 
  ```
  #!/bin/bash

################################# Variables #################################

## Adds current time to var for use in echo for a cleaner log and script ##
DATE=$(date +"%m/%d/%Y %R:%S :")

################################## Script ###################################

echo "$DATE Beginning of Unbind!"

## Load VFIO-PCI driver ##
modprobe vfio
modprobe vfio_pci
modprobe vfio_iommu_type1

#dGPU-VGA  host unbind
echo "1002 73ff"    > /sys/bus/pci/drivers/vfio-pci/new_id
echo "0000:03:00.0" > /sys/bus/pci/devices/0000:03:00.0/driver/unbind
echo "0000:03:00.0" > /sys/bus/pci/drivers/vfio-pci/bind
echo "1002 73ff"    > /sys/bus/pci/drivers/vfio-pci/remove_id

#dGPU-AUDIO host unbind
echo "1002 ab28"    > /sys/bus/pci/drivers/vfio-pci/new_id
echo "0000:03:00.1" > /sys/bus/pci/devices/0000:03:00.1/driver/unbind
echo "0000:03:00.1" > /sys/bus/pci/drivers/vfio-pci/bind
echo "1002 ab28"    > /sys/bus/pci/drivers/vfio-pci/remove_id

echo "$DATE End of Unbind!"
```
4) After you have copied this script, change the Bus ID and Vendor ID to yours (refer to Part -8). 
5) First do the dGPU-VGA, then dGPU-AUDIO
6) Be carefull, you need to change the Bus ID in the file names too, example: "/sys/bus/pci/devices/***0000:03:00.0***/driver/unbind" 
7) Edit */bin/vfio-teardown.sh* to the following: 
```
#!/bin/bash

################################# Variables #################################

## Adds current time to var for use in echo for a cleaner log and script ##
DATE=$(date +"%m/%d/%Y %R:%S :")

################################## Script ###################################

echo "$DATE Beginning of Bind!"


#dGPU host bind

#dGPU-VGA
echo 1 > /sys/bus/pci/devices/0000:03:00.0/remove

#DGPU-AUDIO
echo 1 > /sys/bus/pci/devices/0000:03:00.1/remove
echo 1 > /sys/bus/pci/rescan

sleep "1"

#iGPU host bind

#iGPU-VGA
echo "0000:00:02.0" > /sys/bus/pci/devices/0000:00:02.0/driver/unbind
echo 1              > /sys/bus/pci/devices/0000:00:02.0/remove
echo 1              > /sys/bus/pci/rescan

## Unload VFIO-PCI driver ##
modprobe -r vfio_pci
modprobe -r vfio_iommu_type1
modprobe -r vfio

echo "$DATE End of Bind!"
```
8) After you have copied this script, change the Bus ID and Vendor ID to yours (refer to Part -8). 
9) First do the dGPU-VGA, then dGPU-AUDIO, then iGPU-VGA
10) Be carefull, you need to change the Bus ID in the file names too, example: "/sys/bus/pci/devices/***0000:00:02.0***/driver/unbind" 
 
# Enjoy!


# Know Issues 
1) Black screen after Guest(VM) shutdown, 
   If this is happening, try moving your mouse while the Guest (VM) is shuting down, even through the dark screen until your host starts up (15 to 30 seconds).  

          
          
 

