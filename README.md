# single-amd-gpu-passthrough
Simple, single amd gpu passthrough on intel platform.

We will be using some of the components from risingprismtv/single-gpu-passthrough guide; however, simplified it for plasma-wayland

# Prerequisite
- OS : arch linux
- Knl: 6.6+
- DE : plasma-wayland (only)
- DM : sddm
- dGPU: rdna2  
- CPU: intel 
- VT-d and VT-x capable motherboard 

# Part -1 : Motherboard Bios
1) Enable VT-d and VT-x in your motherboards BIOS 
2) Disable REBAR and 4G Decode 
3) Disable CSM

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
          sudo chown YOURUSERNAME:YOURUSERNAME patched.rom
   ```  
4) Adding the vbios to the VM by following the guide, make sure to use the correct directory: 
https://gitlab.com/risingprismtv/single-gpu-passthrough/-/wikis/8)-Attaching-the-GPU-to-your-VM

# Part -6 : Inputs and Outputs 
Passing USB & Audio Controller  

1) Follow: https://gitlab.com/risingprismtv/single-gpu-passthrough/-/wikis/8)-Attaching-the-GPU-to-your-VM#adding-your-gpu-and-your-rom
2) Instead of passing the GPU, pass-through your PCIE groups(every device in your group) for your Usb and Audio Controller

# Part -7 : Scripts
Scripts to prevent host from sleeping and etc.  

1) Follow the guide: https://gitlab.com/risingprismtv/single-gpu-passthrough/-/wikis/7)-scripts-&-logfiles
2) Edit usr/local/bin/vfio-startup.sh to look like the following:
```
    killall kwin_wayland
    systemctl stop lightdm.service
```
3) Edit usr/local/bin/vfio-teardown.sh to look like the following: 
```
  systemctl restart lightdm.service
```
4) Edit /etc/libvirt/hooks/qemu to change the name of the default VM name to match yours. 


  
# Enjoy!

# Recommendation
1) CPU Performace Tuning,
   https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF#CPU_pinning
2) Potential Anti-Cheat Help, Libvirt Part Only (not tested, use at your own risk)
   https://docs.vrchat.com/docs/using-vrchat-in-a-virtual-machine

# Know Issues 
1) Black screen after Guest(VM) shutdown, 
   As far as I can tell, this is caused by SDDM (the display manager) failing to start properly, use lightdm instead.
   Its better to use https://gitlab.com/risingprismtv/single-gpu-passthrough for x11 based apps (lightdm and plasma) 

          
