# single-amd-gpu-passthrough
simple, single amd gpu passthrough on intel platform

# Prerequisite
- OS : Arch linux
- GPU: 6600xt 
- CPU: 12700k
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
 
   -VGA      (example) 03:00.0 and 1002:73ff  <- dGPU
 
   -AUDIO    (example) 03:00.1 and 1002:ab28  <- dGPU
   
   -VGA      (exmaple) 00:02.0 and 8086:4680  <- iGPU

# Part -3 : Libvirt and Qemu 
Follow the guide: https://gitlab.com/risingprismtv/single-gpu-passthrough/-/wikis/4)-Configuration-of-libvirt

# Part -4 : Virtual Machine (VM)
Follow the guide: https://gitlab.com/risingprismtv/single-gpu-passthrough/-/wikis/5)-Configuring-Virtual-Machine-Manager

# Part -5 : Inputs 
Pass mouse and keyboard to the VM

1) Follow the guide: https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF#Passing_keyboard/mouse_via_Evdev 
2) Navigate to /dev/input/by-id/ directory, here we want to find your the name of your  keyboard and mouse
3) Perform the test provided by the guide and note the device names 
4) Add the folowing to your VM xml file (replace the mouse and keyboard name with what you found) : 
      
      <input type='evdev'>
      <source dev='/dev/input/by-id/MOUSE_NAME'/>
    </input>
    <input type='evdev'>
      <source dev='/dev/input/by-id/KEYBOARD_NAME' grab='all' repeat='on' grabToggle='ctrl-ctrl'/>
    </input>

# Part -6 : Audio 



# Part -7 : GPU VBios
Warning the following may result in permanent DAMAGE, follow at your own RISK!

1) Find your gpu vbios here and Download it: https://www.techpowerup.com/vgabios/
2) Rename the Vbios file to: patched.rom
3) Create a Directory to store and use the vbios:
   '''
          cd ~/Downloads  
          sudo mkdir /var/lib/libvirt/vgabios
          cp ./patched.rom /var/lib/libvirt/vgabios/
          cd /var/lib/libvirt/vgabios/
          sudo chmod -R 660 patched.rom
          sudo chown yourusername:yourusername patched.rom
    '''      
4) Adding the vbios to the VM
5) Follow the guide: https://gitlab.com/risingprismtv/single-gpu-passthrough/-/wikis/8)-Attaching-the-GPU-to-your-VM
6) Directory should be: var/lib/libvirt/vgabios/patched.rom
          
# Part -8 : VM Start-up and Shut-down
          
          
 

