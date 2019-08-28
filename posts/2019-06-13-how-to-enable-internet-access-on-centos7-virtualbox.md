---
layout: post
title: "How to enable internet access of CentOS 7 on VirtualBox"
date: 2019-06-13 23:30:33 +0800
categories: centos, virtualbox
tags: [ centos virtualbox ]
---

I was setting up a CentOS 7 virtual machine using VirtualBox to replicate an
existing setup when I discovered that I cannot access the internet even though
my VM was setup to use a **bridged adapter**.

## Diagnosing the problem
I wanted to know the status of network interfaces on my virtual machine so I
execute the **nmcli** command.
```bash
$ nmcli
enp0s3: disconnected
        "Intel 82540EM"
        1 connection available
        ethernet (e1000), 08:00:27:A7:B4:52, autoconnect, hw, mtu 1500

lo: unmanaged
        "lo"
        loopback (unknown), 00:00:00:00:00:00, sw, mtu 65536

Use "nmcli device show" to get complete information about known devices and
"nmcli connection show" to get an overview on active connection profiles.

Consult nmcli(1) and nmcli-examples(5) manual pages for complete usage details.
```

The command shows that enp0s3 is diconnected so I checked its network
configuration script.
```bash
$ cat /etc/sysconfig/network-scripts/ifcfg-enp0s3
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=dhcp
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=enp0s3
UUID=1056c9af-dc47-429b-958c-d0e83b99d60f
DEVICE=enp0s3
ONBOOT=no
```
I found out that the reason I have no access to the internet is because it is
not configured to be activated on boot. The last line **ONBOOT=no** means that
this network interface won't be activated on startup.

## Configuring the network interface

To resolve this I opened a text editor to edit the network configuration and
changed the last line to **ONBOOT=yes**.
```bash
sudo vi /etc/sysconfig/network-scripts/ifcfg-enp0s3
```
I have to restart the network and apply the changes.
```bash
sudo systemctl restart network.service
```
Now if I check the interface, it would show that it is connected.

```bash
$ nmcli
enp0s3: connected to enp0s3
        "Intel 82540EM"
        ethernet (e1000), 08:00:27:A7:B4:52, hw, mtu 1500
        ip4 default
        inet4 10.0.2.15/24
        route4 0.0.0.0/0
        route4 10.0.2.0/24
        inet6 fe80::f63f:8611:805b:579d/64
        route6 fe80::/64
        route6 ff00::/8

lo: unmanaged
        "lo"
        loopback (unknown), 00:00:00:00:00:00, sw, mtu 65536

DNS configuration:
        servers: 17.83.200.200 17.83.200.202
        domains: apple.com
        interface: enp0s3

Use "nmcli device show" to get complete information about known devices and
"nmcli connection show" to get an overview on active connection profiles.

Consult nmcli(1) and nmcli-examples(5) manual pages for complete usage details.
```

## Testing access to the internet
Now to check if my virtual machine has access to the internet.
```bash
$ ping google.com
PING google.com (74.125.24.138) 56(84) bytes of data.
64 bytes from 74.125.24.138 (74.125.24.138): icmp_seq=1 ttl=41 time=71.7 ms
64 bytes from 74.125.24.138 (74.125.24.138): icmp_seq=2 ttl=41 time=73.8 ms
64 bytes from 74.125.24.138 (74.125.24.138): icmp_seq=3 ttl=41 time=73.1 ms
```

The command above shows that my virtual machine can now successfully connect to
the internet.
