---
layout: post
title: PCI Passthrough On Centos 7
disqus: true
---
Thanks to the vfio feature of the new linux kernel and OVMF, we can now do PCI-Passthrough much easier than before. I tried it days ago just following this [wiki][1]. I thought it would be hard and bumpy, but it turns out that I succeed on the very first time. 
here’s my enviroments:

> CPU: Intel(R) Xeon(R) CPU E5-2620 v3 @ 2.40GHz
> Graphic Card: NVIDIA Qurado M2000
> OS: Centos 7
> Kernel: 4.9.2-1.el7.elrepo.x86\_64

and here’s my libvirt script:

```xml
<domain type='kvm' xmlns:qemu='http://libvirt.org/schemas/domain/qemu/1.0'>
        <name>win8</name>
        <memory>16777216</memory>
        <currentMemory>16777216</currentMemory>
        <vcpu>4</vcpu>
        <os>
          <type arch='x86_64' machine='pc'>hvm</type>
	  <loader readonly='yes' type='pflash'>/usr/share/edk2.git/ovmf-x64/OVMF_CODE-pure-efi.fd</loader>
	   <nvram>/var/lib/libvirt/qemu/nvram/win8_VARS.fd</nvram>
          <boot dev='cdrom'/>
          <boot dev='hd'/>
       </os>
       <features>
         <acpi/>
         <apic/>
         <pae/>
	<kvm>
	   <hidden state='on'/>
	</kvm>
       </features>
       <clock offset='localtime'/>
       <on_poweroff>destroy</on_poweroff>
       <on_reboot>restart</on_reboot>
       <on_crash>destroy</on_crash>
       <devices>
         <emulator>/usr/libexec/qemu-kvm</emulator>
	  <disk type='file' device='cdrom'>
           <source file='/root/img/cn_windows_8_x64_dvd_915407.iso'/>
           <target dev='hda' bus='ide'/>
         </disk>
         <disk type='file' device='disk'>
          <driver name='qemu' type='qcow2'/>
           <source file='/root/img/win8.qcow2'/>
           <target dev='vda' bus='virtio'/>
         </disk>
	  <disk type='file' device='cdrom' >
           <source file='/root/img/virtio-win-0.1.126.iso'/>
           <target dev='hdb' bus='ide'/>
         </disk>
        <interface type='bridge'>
          <source bridge='br0' />
	  <model type='virtio' />
        </interface>
        <input type='tablet' bus='usb' /> 
	<hostdev mode='subsystem' type='pci' managed='yes'>
     		<driver name='vfio' multifunction='on'/>
     		<source>
      			 <address type='pci' domain='0x0000' bus='0x82' slot='0x00' function='0x0'/>
     		</source>
	</hostdev>
<hostdev mode='subsystem' type='usb' managed='yes'>
        <source>
                <vendor id='0x0101'/>
                <product id='0x0007'/>
        </source>
</hostdev>
<hostdev mode='subsystem' type='usb' managed='yes'>
        <source>
                <vendor id='0x5c0a'/>
                <product id='0x8502'/>
        </source>
</hostdev>
<graphics type='vnc' port='-1' autoport='yes' listen = '0.0.0.0' keymap='en-us'/>
        <video>
                <model type='cirrus' vram='16384' heads='1'/>
        </video>
       </devices>
     </domain>
```

The most weird and wonderful thing is, almost all documents I could find on the internet indicate that I can not install windows 7 or windows server 2008 on a **pure uefi** bios which is needed by PCI-Passthrough. But I succeed even without OVMF, I guess maybe I take credit for working on a very very new linux kernel(4.9). Here’s the script:

```xml
<domain type='kvm'>
        <name>win7</name>
        <memory>16777216</memory>
        <currentMemory>16777216</currentMemory>
        <vcpu>12</vcpu>
        <cpu mode='host-passthrough'/>
        <os>
          <type arch='x86_64' machine='pc'>hvm</type>
          <boot dev='cdrom'/>
          <boot dev='hd'/>
       </os>
       <features>
         <acpi/>
         <apic/>
         <pae/>
       </features>
       <clock offset='localtime'/>
       <on_poweroff>destroy</on_poweroff>
       <on_reboot>restart</on_reboot>
       <on_crash>destroy</on_crash>
       <devices>
         <emulator>/usr/libexec/qemu-kvm</emulator>
	<disk type='file' device='cdrom' index='0'>
           <source file='/root/img/cn_windows_7_professional_with_sp1_vl_build_x64_dvd_u_incl_virtio-140506-homemade-by-Jetso.iso'/>
           <target dev='hda' bus='ide'/>                                                     
         </disk> 
         <disk type='file' device='disk' index='1'>
          <driver name='qemu' type='qcow2'/>
           <source file='/home/img/win7.qcow2'/>
           <target dev='vda' bus='virtio'/>
         </disk>
	  <disk type='file' device='cdrom' index='2' >
           <source file='/root/img/virtio-win-0.1.126.iso'/>
           <target dev='hdb' bus='ide'/>
         </disk>
        <interface type='bridge'>
          <source bridge='br0' />
          <model type='virtio' />
        </interface>
        <input type='tablet' bus='usb' /> 
	<hostdev mode='subsystem' type='pci' managed='yes'>
     		<driver name='vfio'/>
     		<source>
      			 <address type='pci' domain='0x0000' bus='0x82' slot='0x00' function='0x0'/>
     		</source>
	</hostdev>
<hostdev mode='subsystem' type='usb' managed='yes'>
        <source>
                <vendor id='0x0101'/>
                <product id='0x0007'/>
        </source>
</hostdev>
<hostdev mode='subsystem' type='usb' managed='yes'>
        <source>
                <vendor id='0x5c0a'/>
                <product id='0x8502'/>
        </source>
</hostdev>
<graphics type='vnc' port='-1' autoport='yes' listen = '0.0.0.0' keymap='en-us'/>
        <video>
                <model type='cirrus' vram='16384' heads='1'/>
        </video>
       </devices>
     </domain>
```




[1]:	https://wiki.archlinux.org/index.php/PCI_passthrough_via_OVMF