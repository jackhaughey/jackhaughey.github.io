# NMAP in a Distrobox Container

Trying to use Nmap in my SilverBlue Linux laptop, by way of a distrobox container when I get this error;

```
sudo nmap -sS 10.2.0.90
Starting Nmap 7.92 ( https://nmap.org ) at 2026-10-21 08:18 BST
dnet: Failed to open device wlp0s20f3
QUITTING!
```

This occurs because Nmap is trying to use raw socket packet injection (-sS SYN stealth scan), but my rootless Distrobox container cannot properly bind or maps raw sockets directly to the host's physical wireless adapter, wlp0s20f3.

To resolve this, I created a "rootfull" container;

```
distrobox create --name nmap-box --image ubuntu:latest --root
```

Am in two minds as to add more tools to this container, or limit it to nmap.
