# Troubleshooting SSH Authentication After Rebuilding Rocky Linux Kubernetes Nodes

While rebuilding the Kubernetes portion of my homelab, I encountered an unexpected SSH issue that initially appeared to be related to FreeIPA, Ansible or SSH key distribution.

The environment consists of:

RHEL 10 administration server
RHEL 10 storage server
Rocky Linux Identity Management (FreeIPA) server
Rocky Linux Kubernetes nodes

SSH key-based authentication was already working correctly between the administration, storage and IPA servers. However, the newly built Kubernetes nodes continued to prompt for a password despite the SSH public key being present in the authorized_keys file.

Symptoms

From the administration server:
```
ssh-copy-id k8s-control-01
```

completed successfully and confirmed the SSH public key had been added.

Inspecting the target host showed:
```
cat ~/.ssh/authorized_keys
```

contained the expected public key.

However:

```
ssh k8s-control-01
```
would still prompt for a password.

## Initial Investigation

Several common causes were investigated:

Incorrect SSH permissions
Incorrect SELinux contexts
SSSD configuration
FreeIPA client configuration
SSH daemon configuration

Everything appeared correct.

The breakthrough came from reviewing the SSH daemon logs:

```
sudo journalctl -f -u sshd
```

which showed;
```
userauth_pubkey: signature algorithm ssh-ed25519 not in PubkeyAcceptedAlgorithms
```

## Root Cause

Further investigation showed the Kubernetes nodes had been installed using the Rocky Linux FIPS profile rather than the standard installation profile.

Checking the active crypto policy:
```
sudo update-crypto-policies --show
```

returned

```
FIPS
```

and

```
cat /proc/sys/crypto/fips_enabled
```

returned;
```
1
```

The kernel command line also contained:
```
fips=1
```

indicating the systems were booting in full FIPS mode.

Under FIPS mode, the ssh-ed25519 algorithm used by my administration server was not permitted, causing SSH public key authentication to fail despite the key being correctly installed.

## Resolution

The issue was traced back to an installation mistake rather than a FreeIPA or SSH configuration issue.

At this point in the build process, the Kubernetes nodes had only been:

Installed
Assigned static IP addresses
Configured with DNS
Joined to FreeIPA

Since no Kubernetes components had yet been installed, rebuilding the nodes using the standard Rocky Linux installation profile was the cleanest solution.

## Lessons Learned

This was a useful reminder that authentication failures are not always caused by the obvious components.

What initially looked like:
```
SSH issue
```

turned out to be a;
```
Crypto policy issue
```

introduced during operating system installation.

More importantly, it highlighted how security policies can affect seemingly unrelated services. Reviewing SSH daemon logs quickly revealed the real problem and prevented a significant amount of unnecessary troubleshooting.

## Takeaway

Before beginning Kubernetes installation and Ansible automation, it is worth verifying:
```
update-crypto-policies --show
```

and;
```
cat /proc/sys/crypto/fips_enabled
```

to ensure systems are operating with the expected cryptographic policies.

In this case, a simple installation choice had a direct impact on SSH authentication and infrastructure automation.

## Next Step: 

Rebuild the Kubernetes nodes using the standard Rocky Linux profile and continue with Ansible-driven configuration management before deploying containerd, kubeadm and the Kubernetes control plane.
