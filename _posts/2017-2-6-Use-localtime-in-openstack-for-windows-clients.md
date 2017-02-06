---
layout: post
title: Use localtime in openstack for windows clients
---

Recently we came across a issue in which a customer complained his windows client host timezone was always wrong. There's an 8 hours delay compares to the localtime(our customer were in China which uses UTC +8). Obviousely, it's an timezone problem. After googling and reading the source code of nova, I found something connected with this issue:

[A question](https://ask.openstack.org/en/question/66649/instance-time-wrong/)

[A Bug](https://bugs.launchpad.net/nova/+bug/1231254)

and the code in _nova/virt/libvirt/driver.py_

```python
def _set_clock(self, guest, os_type, image_meta, virt_type):
        # NOTE(mikal): Microsoft Windows expects the clock to be in
        # "localtime". If the clock is set to UTC, then you can use a
        # registry key to let windows know, but Microsoft says this is
        # buggy in http://support.microsoft.com/kb/2687252
        clk = vconfig.LibvirtConfigGuestClock()
        if os_type == 'windows':
            LOG.info(_LI('Configuring timezone for windows instance to '
                         'localtime'))
            clk.offset = 'localtime'
        else:
            clk.offset = 'utc'
        guest.set_clock(clk)
        if virt_type == "kvm":
            self._set_kvm_timers(clk, os_type, image_meta)
```

so, how to solve the problem? apprently there are two scenarios you may run into:

## 1st. Set up at the very beginning

Just create create the image with a property os_type=windows like this:

```
glance image-update --property os_type="windows" <IMAGE-ID>
```

## 2nd. Modify after instance launched

If you've launched the instance without a os_type set before, you could hack the database manually to add a os_type for it:

```
# mysql -u root -p
$ use nova;
$ UPDATE instances set os_type='windows' where hostname='uhrzeit-test';
$ select hostname,os_type from instances WHERE hostname='uhrzeit-test';
+--------------+---------+
| hostname     | os_type |
+--------------+---------+
| uhrzeit-test | windows |
+--------------+---------+
```

Then reboot your VM(hard reboot).
