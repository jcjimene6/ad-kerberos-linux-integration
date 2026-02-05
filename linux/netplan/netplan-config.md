# Netplan Configuration and Renderer Selection

## Overview

This document describes the static networking configuration on the Linux host and the issues encountered due to netplan renderer selection and overlapping network management systems.

Understanding this behavior was critical to resolving connectivity issues later in the lab.

---

## Initial Static Configuration

After installation, the Linux host was configured with a static IP address and DNS settings using netplan.

The initial configuration file was edited at:

/etc/netplan/00-installer-config.yaml


Example configuration:

---
```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: no
      addresses:
        - 172.16.0.50/24
      gateway4: 172.16.0.1
      nameservers:
        addresses:
          - 172.16.0.10
        search:
          - mydomain.com
```

The configuration was applied using:

sudo netplan generate
sudo netplan apply

## Unexpected Behavior

Although the configuration appeared correct:
- The NIC was detected by the kernel
- The interface came up
- IPv6 addressing appeared active

IPv4 connectivity did not function as expected.

Running ip link showed the interface, but IPv4 traffic was unreliable.

---

## Root Cause: Multiple Networking Authorities

On Ubuntu Server, networking involves multiple layers:

1. **cloud-init**  
   Performs initial provisioning and records early network assumptions.

2. **netplan**  
   Acts as a configuration translator, generating backend-specific configuration.

3. **Renderer (systemd-networkd or NetworkManager)**  
   Brings interfaces up and applies the configuration.

In this case:
- cloud-init had already established assumptions during installation
- netplan was generating configuration
- the active renderer was not consistently applying it

As a result, two systems believed they were responsible for networking, and neither fully controlled the interface.

---

## Renderer Explicitly Defined

To resolve the ambiguity, the netplan configuration was updated to explicitly specify the renderer:

---
```yaml
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      dhcp4: no
      addresses:
        - 172.16.0.50/24
      nameservers:
        addresses:
          - 172.16.0.1
        search:
          - mydomain.com
```

After reapplying the configuration, IPv4 connectivity behaved consistently.

## Validation

Connectivity was validated by performing the following checks:

- Verified reachability of the domain controller using ICMP
- Confirmed DNS resolution using `resolvectl`
- Ensured the interface remained up after applying netplan changes

At this stage, the Linux host had stable internal network connectivity and could reliably communicate with the domain controller.

## Key Takeaway

When static networking behaves inconsistently on Ubuntu Server, the issue may not be the configuration itself, but ambiguity between multiple systems attempting to manage the same interface.

Explicitly defining the netplan renderer ensures that a single backend is responsible for applying network configuration and prevents conflicting behavior between cloud-init, netplan, and the renderer.
