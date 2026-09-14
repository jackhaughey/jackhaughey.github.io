# Juniper vMX device first start
On first booting a vMX Router, I was getting a lot of DHCP messages;
```
Auto Image Upgrade.
```

To disable them, I ran;
```
root@:~ # cli
root> configure
[edit]
root# delete chassis auto-image-upgrade
[edit]
root# commit
```
