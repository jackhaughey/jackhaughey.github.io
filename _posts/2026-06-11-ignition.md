# Ignition 
.. is a first‑boot, declarative provisioning system used by Fedora CoreOS, RHEL CoreOS, and Flatcar.

-    Runs once at first boot
-    Applies the entire machine state
-    Designed for immutable OSes
-    Configuration is JSON/YAML → compiled → delivered at boot
-    Cannot be re-run after boot
-    Perfect for GitOps, reproducibility, Kubernetes nodes, atomic updates

Ignition is basically “infrastructure as code for the OS image”.
