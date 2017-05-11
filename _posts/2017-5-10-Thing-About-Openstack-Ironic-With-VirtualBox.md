---
layout: post
title: Things about Openstack ironic with virtualbox
---
It's a tough work to try ironic with virtualbox. I finally make it work after 3 days of hardwork. Here's some issue that I met and solved.

## Enviroment

host: 

> CPU: Intel(R) Xeon(R) CPU E5-2620 v2 @ 2.10GHz
>
> MEM: 64GB
>
> OS: Centos 7.3

I create 2 kvm virtual machine, one for openstack, one for virtualbox.

| node             | cpu     | memory | ip          | os            |
| ---------------- | ------- | ------ | ----------- | ------------- |
| openstack mitaka | 6 cores | 16 GB  | 10.0.40.166 | centos7.3     |
| virtualbox       | 6 core2 | 16 GB  | 10.0.40.167 | windows 7 x64 |

You could install a virtualbox on linux like ubuntu, but I strongly recommended to use a windows version. Linux version virtualbox is not so stable. It always got stucked or crash during my test.

## 1. Virtualbox got stuck and not response

Use a windows version. Linux version is not stable, it'll crash or get stucked after several times.

## 2. VBoxWebSrv shows Permission denied for ''

Turn off validation for vboxwebsrv:

```VBoxManage setproperty websrvauthlibrary null```

## 3. VirtualBox machine 'DHCP Connection timeout'

![](/images/dhcp_timeout.png)

first check the link between virtualbox vm and neutron dhcp service:

find dhcp service ip address, then ping from virtualbox vm to this ip address.

```
[root@node01 images]# ip netns exec qdhcp-f0949907-5de7-465c-b448-2abd4448a0ca ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
14: tap95e88e2b-d5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN qlen 1000
    link/ether fa:16:3e:41:93:01 brd ff:ff:ff:ff:ff:ff
    inet 10.0.40.210/24 brd 10.0.40.255 scope global tap95e88e2b-d5
       valid_lft forever preferred_lft forever
    inet 169.254.169.254/16 brd 169.254.255.255 scope global tap95e88e2b-d5
       valid_lft forever preferred_lft forever
    inet6 fe80::f816:3eff:fe41:9301/64 scope link 
       valid_lft forever preferred_lft forever
```

if the link is ok, go on check neutron port

```
[root@node01 images]# openstack port list
+--------------------------------------+------+-------------------+---------------------------------------------------+
| ID                                   | Name | MAC Address       | Fixed IP Addresses                                |
+--------------------------------------+------+-------------------+---------------------------------------------------+
| 11bccde1-c21d-4a8e-a8ad-9353467716b8 |      | 08:00:27:2c:33:7f | ip_address='10.0.40.219', subnet_id='c4f8639f-    |
|                                      |      |                   | ce8e-431a-832d-ca80f32f7dc6'                      |
| 95e88e2b-d52c-498b-b7cd-39c0fb70a8b6 |      | fa:16:3e:41:93:01 | ip_address='10.0.40.210', subnet_id='c4f8639f-    |
|                                      |      |                   | ce8e-431a-832d-ca80f32f7dc6'                      |
+--------------------------------------+------+-------------------+---------------------------------------------------+
```

There will be a port has the same mac address as your virtualbox vm.

## 4.pxelinux.0 no such file or directory 

![](/images/pxe0.png)

Check you tftp service on openstack node first. Then check if the file `pxelinux.0` exists in that directory.

Ironic use `/tftproot/` as tftp directory by default, you could copy all pxelinux related files to that directory:

```cp /var/lib/tftpboot/* /tftpboot/```

or just change ironic configurations:

```ini
[pxe]
tftp_root = /var/lib/tftpboot/
tftp_master_path = /var/lib/tftpboot/master_images
```

## 5. How to clear the error state of ironic

if you failed to boot a baremetal instance, the ironic node will stuck at `clear failed`, `clean wait` or `failed` provision state. 

```provision_state        | clean wait```

You must change the provision state to `avaliable` before you could begin another try. Unfortuanately, there's no official way to do that. You need to modify the mysql record manually. Here's a clear script works fine to me:

```
#!/bin/bash
echo 'clearing up...'
nova delete test
mysql -D ironic  -e 'UPDATE nodes SET provision_state="available", target_provision_state=NULL;'
ironic node-set-power-state $NODE_UUID off
ironic node-set-maintenance $NODE_UUID false
```

then find the neutron port related to your vm and delete it, or the openstack-nova-compute will fail to create it again.





####  

