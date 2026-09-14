# Juniper vMX device first start
On first booting a vMX Router, I was getting a lot of DHCP messages;
```
Auto Image Upgrade.
```

To disable them, I ran;
```
configure
delete chassis auto-image-upgrade
commit
```
