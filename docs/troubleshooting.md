# Troubleshooting

This page recap the possible actions to realize if you have any problems with your XOA.

## Empty page after login

This is happening when your anti-virus or firewall is blocking websocket protocol. This is what we use to communicate between `xo-server` and `xo-web` (see the [architecture page](architecture.md)).

The solution is to use **HTTPS**. In this way, websockets will be encapsulated in the secured protocol, avoiding interception from your firewalls or anti-virus system.

## XOA configuration

XOA is a virtual appliance running Debian and Xen Orchestra. If you have any problem, the first thing to do is to check the network configuration.

The easiest step is to check if you can ping our servers:

```
$ ping xen-orchestra.com

PING xen-orchestra.com (*******) 56(84) bytes of data.
64 bytes from xen-orchestra.com (*******): icmp_seq=1 ttl=53 time=11.0 ms
64 bytes from xen-orchestra.com (*******): icmp_seq=2 ttl=53 time=11.0 ms
```

If you have something completely different than that, or error messages, lost packets etc., it means you have probably a network problem. Check the following sections on network and DNS configuration.

### IP configuration

You can see your current network configuration with a `ifconfig eth0`. If you have a firewall, please check that you allow the XOA's IP.

You can modify the IP configuration in `/etc/network/interfaces`.

[Follow the official Debian documentation about that]( https://wiki.debian.org/NetworkConfiguration#Configuring_the_interface_manually).


### DNS configuration

The DNS servers are configured in `/etc/resolv.conf`. If you have problems with DNS resolution, please modify the IPs of those servers to fit your current network configuration.

[Check the official Debian documentation to configure it](https://wiki.debian.org/NetworkConfiguration#The_resolv.conf_configuration_file).

### Disk space

You can run `df -h` to check if you don't have space disk issue. If you want to backup your VMs in XOA, [take a look here](https://xen-orchestra.com/docs/full_backups.html#add-a-disk-for-local-backups).

### Behind a transparent proxy

If your are behind a transparent proxy, you'll probably have issues with the updater (SSL/TLS issues).

First, run the following commands:

```
$ echo NODE_TLS_REJECT_UNAUTHORIZED=0 >> /etc/xo-appliance/env
$ npm config -g set strict-ssl=false
```

Then, restart the updater with `systemctl restart xoa-updater`.

## XO configuration

The system logs are visible thanks to this command:

```
$ tail -f /var/log/syslog
```

You can read more about logs [in the dedicated chapter](logs.md).

### Reset XO configuration

If you have problems with your `xo-server` configuration, you can reset the database. **This operation will delete all your configured users and servers**:

1. `redis-cli`
2. `FLUSHALL`
3. `systemctl restart xo-server.service`

You can now log in with `admin@admin.net` and `admin` password.

### Redownload and rebuild all the packages

If a package disappear due to a build problem or a human error, you can redownload them using the updater:

1. `rm /var/lib/xoa-updater/update.json`
2. `xoa-updater --upgrade`

> We'll have a `xoa-updater --force-reinstall` option soon, to do this automatically
