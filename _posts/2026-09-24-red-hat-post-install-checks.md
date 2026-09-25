# Red Hat Developer post installation checks
I recently had an issue where the repo file was empty. I assume that there was some kind of "blip" when registering the system.
As a result, I now run these check;

```
sudo subscription-manager register
sudo subscription-manager refresh
sudo dnf repolist
sudo dnf update
```

To fix the original problem, I un-registered, and then re-registered;
```
sudo subscription-manager unregister
Unregistering from: subscription.rhsm.redhat.com:443/subscription
System has been unregistered.

[jack@localhost ~]$ sudo subscription-manager clean
All local data removed

[jack@localhost ~]$ sudo rm -f /etc/yum.repos.d/redhat.repo

[jack@localhost ~]$ sudo subscription-manager register
Registering to: subscription.rhsm.redhat.com:443/subscription
Username: jhaughey
Password: 
The system has been registered with ID: 772d89ad-3202-4bf5-98e1-10f969b59d48
The registered system name is: localhost.localdomain
```
