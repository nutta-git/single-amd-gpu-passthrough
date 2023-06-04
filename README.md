# single-amd-gpu-passthrough
Simple, single amd gpu passthrough on intel platform (only).

We will be using some of the components from risingprismtv/single-gpu-passthrough guide; however, simplified it for plasma-wayland

# Prerequisite
- OS : Arch linux
- Knl: linux-zen
- DE : Plasma-Wayland (only)
- DM : sddm-git (required) 
- dGPU: RNDA2  
- CPU: 12th-gen(K) Intel 
- VT-d and VT-x capable motherboard 

# Part -1 : Motherboard Bios
1) Enable VT-d and VT-x in your motherboards BIOS 
2) Disable REBAR and 4G decode 
3) Disable CSM
4) Disbale iGPU

# Part -2 : Bootloader and IOMMU 
1) Add the following parameters to your bootloader: ```intel_iommu=on iommu=pt```
2) Restart 
3) Follow the guide: https://gitlab.com/risingprismtv/single-gpu-passthrough/-/wikis/3)-IOMMU-Groups 
  
  Note down the Bus ID for GPU: 
 ```
   -VGA      (example) 03:00.0 
   -AUDIO    (example) 03:00.1 
```
# Part -3 : Libvirt and Qemu 
1) Follow the guide: https://gitlab.com/risingprismtv/single-gpu-passthrough/-/wikis/4)-Configuration-of-libvirt

# Part -4 : Virtual Machine (VM)
1) Follow the guide: https://gitlab.com/risingprismtv/single-gpu-passthrough/-/wikis/5)-Configuring-Virtual-Machine-Manage

# Part -5 : dGPU VBios and Binding
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
4) Adding the vbios to the VM by following the guide: 
https://gitlab.com/risingprismtv/single-gpu-passthrough/-/wikis/8)-Attaching-the-GPU-to-your-VM

# Part -6 : Inputs and Outputs 
Passing USB & Audio Controller  

1) Follow: https://gitlab.com/risingprismtv/single-gpu-passthrough/-/wikis/8)-Attaching-the-GPU-to-your-VM#adding-your-gpu-and-your-rom
2) Instead of passing the GPU, pass through your PCIE groups(every device in your group) that contains your Usb Controller and your Audio Controller

# Part -7 : Disable Sleep and Suspend
Prevent Host from sleeping or suspending

1) With Plasma open *System-Settings*
2) Travel to *Power Management --> Engergy Saving* 
3) Uncheck *Suspend session*
  
# Enjoy!

# Know Issues 
1) Black screen after Guest(VM) shutdown, 
   If this is happening, try moving your mouse while the Guest (VM) is shuting down, even through the dark screen until your host starts up (15 to 30 seconds). 
          
