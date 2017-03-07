---
layout: post
title: GPU-Passthrough-With-VFIO
---
In one of my previous page, I wrote about how to do GPU Passthrough with VFIO. It contains too much principles and meaningless words. To make it simple and clear, I record the bare steps I did to passthrough a NVidia graphic card in a CentOS 7.3 system.

1. Find out the vendorid and productid of the graphic card

   ```bash
   [root@fusion-node01 ~]# lspci -nn | grep -i vga
   0a:00.0 VGA compatible controller [0300]: Matrox Electronics Systems Ltd. G200eR2 [102b:0534] (rev 01)
   82:00.0 VGA compatible controller [0300]: NVIDIA Corporation GM206GL [Quadro M2000] [10de:1430] (rev a1)
   ```

   group: 82

   venderid: 10de

   productid: 1430

2. Modify the grub parameters and modprobe list

   add **rd.driver.pre=vfio-pci intel_iommu=on** 

   ```ini
   [root@fusion-node01 ~]# cat /etc/sysconfig/grub 
   GRUB_TIMEOUT=5
   GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
   GRUB_DEFAULT=saved
   GRUB_DISABLE_SUBMENU=true
   GRUB_TERMINAL_OUTPUT="console"
   GRUB_CMDLINE_LINUX="crashkernel=auto rd.lvm.lv=cl/root rd.lvm.lv=cl/swap rhgb quiet rd.driver.pre=vfio-pci intel_iommu=on"
   GRUB_DISABLE_RECOVERY="true"
   ```

   add driver blacklist for nvidia

   ```
   [root@fusion-node01 ~]# cat /etc/modprobe.d/blacklist.conf 
   blacklist nouveau
   blacklist nvidia
   ```

   add vfio options 

   ```
   [root@fusion-node01 ~]# cat /etc/modprobe.d/vfio.conf 
   options vfio-pci ids=10de:1430,10de:0fba
   ```

   add modprobe modules

   ```
   [root@fusion-node01 ~]# cat /etc/modules-load.d/vfio.conf 
   vfio 
   vfio_iommu_type1 
   vfio_pci 
   ```

3. Regenerate the grub configuration and initramfs

   ```
   sudo grub2-mkconfig -o /boot/efi/EFI/centos/grub.cfg
   ```

   ```
   mv /boot/initramfs-$(uname -r).img /boot/initramfs-$(uname -r).img.bak
   dracut -v /boot/initramfs-$(uname -r).img $(uname -r)
   ```

   ​

4. Reboot and make sure the vfio driver is in effect

   ```
   dmesg | grep vfio
   [    0.000000] Command line: BOOT_IMAGE=/vmlinuz-3.10.0-514.el7.x86_64 root=/dev/mapper/cl-root ro crashkernel=auto rd.lvm.lv=cl/root rd.lvm.lv=cl/swap rhgb quiet rd.driver.pre=vfio-pci intel_iommu=on
   [    0.000000] Kernel command line: BOOT_IMAGE=/vmlinuz-3.10.0-514.el7.x86_64 root=/dev/mapper/cl-root ro crashkernel=auto rd.lvm.lv=cl/root rd.lvm.lv=cl/swap rhgb quiet rd.driver.pre=vfio-pci intel_iommu=on
   [   12.356671] vfio_pci: add [10de:1430[ffff:ffff]] class 0x000000/00000000
   [   12.356712] vfio_pci: add [10de:0fba[ffff:ffff]] class 0x000000/00000000
   ```

   make sure you see the ```vfio_pci: add``` infomation.

5. Create a vm using the passthroughed vga card as hostdev

   ```xml
   <hostdev mode='subsystem' type='pci' managed='yes'>
                   <driver name='vfio'/>
                   <source>
                            <address type='pci' domain='0x0000' bus='0x82' slot='0x00' function='0x0'/>
                   </source>
           </hostdev>
   ```




https://gist.github.com/cuibonobo/d354440fecdd37c35ecd

https://wiki.archlinux.org/index.php/PCI_passthrough_via_OVMF

https://forum.level1techs.com/t/gta-v-on-linux-skylake-build-hardware-vm-passthrough/87440

http://www.firewing1.com/howtos/fedora-20/create-gaming-virtual-machine-using-vfio-pci-passthrough-kvm

https://wiki.debian.org/VGAPassthrough