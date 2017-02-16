---
layout: post
title: Five Things To Speed Up Openstack Dashboard
---
Horizon is the dashboard project of openstack. It's good, but not very fast. Without any optimization, it can easily take you up to 8-10s to list instances or list volumes. It's ok on a test enviroment. But sometimes it's unacceptable when you have to demonstrate it to the others say your boss or customer.

There's 5 things you can do to speed up the performance of horizon a little bit.

### 1. Use memcache

Memcache brings an obviouse improvement.

/etc/openstack-dashboard/local_settings

```python
CACHES = {
    'default': {
         'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
         'LOCATION': ['node1:11211','node2:11211']
    }
}
```

### 2. Enable COMPRESS_OFFLINE

I'm not pretty sure if this will take any effect, but someone says it brings down the response time from 8s to 3s on his enviroment.

/etc/openstack-dashboard/local_settings

```python
COMPRESS_OFFLINE = True
```

### 3. Use 'INFO' log level

Horizon use 'DEBUG' level on default. Change it to 'INFO'.

/etc/openstack-dashboard/local_settings

```python
 'horizon.operation_log': {
            'handlers': ['operation'],
            'level': 'INFO',
            'propagate': False,
        },
        'openstack_dashboard': {
            'handlers': ['console'],
            'level': 'INFO',
            'propagate': False,
        },
```

### 4. Increase process and threads number of horizon 

/etc/httpd/conf.d/openstack-dashboard.conf

```python
WSGIDaemonProcess dashboard processes=5 threads=15
```



### 5. Use memcache in keystone

 /etc/keystone/keystone.conf

```ini
[cache]
enabled = true
memcached_servers = node1:11211,node2:11211,node3:11211
backend = oslo_cache.memcache_pool
```



Then restart httpd/apache and you'll happy to see some changes.