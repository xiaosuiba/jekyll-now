---
layout: post
title: Disable multipathing with ovirt
---
If you have ovirt engine installed on your host server, it'll automatically configurate multipathing for you. The config file /etc/multipath.conf will be replaced at every boot time by vdsm. This would cause trouble sometimes.

Luckly there's a way to blacklist this feature which I found from a [mail thread](http://lists.ovirt.org/pipermail/users/2015-May/033062.html). Just 2 steps.

1. Insert this code in the second line of your ```/etc/multipath.conf```

```
# RHEV PRIVATE
blacklist {
        devnode "*"
}
```

this will tell vdsm not to cover this file will its settings.

2. turn off multipath manually or just reboot.

```
multipath -d -v3
```

Now you don't have multipathing enabled in your system anymore.