---
layout: post
title: Enable Nested KVM on Centos7.3
---
# 1. Turn on nested KVM temporarily

```bash
echo 'options kvm_intel nested=1' > /etc/modprobe.d/kvm-nested.conf
modprobe -r kvm_intel
modprobe kvm_intel
cat /sys/module/kvm_intel/parameters/nested 
Y
```

# 2. Turn on nested KVM permantely

```bash
echo 'options kvm_intel nested=1' > /etc/modprobe.d/kvm-nested.conf
dracut -v /boot/initramfs-$(uname -r).img $(uname -r) --force
reboot
cat /sys/module/kvm_intel/parameters/nested 
Y
```



