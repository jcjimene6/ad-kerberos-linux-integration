# Linux Networking Failure Caused by cloud-init and NIC Topology Changes

## Symptoms

While attempting to install packages on the Linux host, basic network operations failed:

- `apt update` could not reach external repositories
- Errors indicated no route to the internet
- IPv4 connectivity appeared partially functional
- DNS resolution behaved inconsistently

Example error:
Cannot initiate the connection to security.ubuntu.com:80
Network is unreachable


At first glance, routing tables and IP configuration appeared correct.

---

## Initial Observations

Two separate issues were present:

1. The Linux host did not have a reliable route to the internet
2. Package management attempted IPv6 first, masking the real failure

However, neither of these explained why routing *looked* correct while traffic never left the host.

---

## Root Cause

The core issue was a mismatch between Linux’s internal view of the network and the hypervisor’s actual NIC topology.

During installation, Ubuntu Server uses **cloud-init** to perform initial provisioning. At install time, cloud-init:

- Detects available NICs and their MAC addresses
- Determines which interfaces have DHCP
- Writes a persistent network definition (`50-cloud-init.yaml`)
- Treats this definition as authoritative

This model assumes a cloud environment where network interfaces never change.

---

## What Went Wrong

1. **Initial installation**
   - The VM was installed with only an internal NIC
   - No default gateway or external connectivity existed
   - cloud-init recorded this state as the “correct” network layout

2. **NIC topology changed later**
   - A NAT/external NIC was added
   - Interfaces were reordered and attachment types changed
   - MAC-to-interface assumptions no longer matched reality

3. **Resulting failure mode**
   - The Linux kernel detected interfaces correctly
   - Netplan applied configuration without errors
   - Routing tables appeared valid
   - DHCP intermittently worked
   - IPv4 addresses were assigned

However, adjacency at Layer 2 was broken.

Linux sent ARP requests such as:

Who has 10.0.2.2?


The VirtualBox NAT backend never responded because the NIC identity no longer matched its expectations.

ARP entries remained in a FAILED state, and Linux immediately returned:

Destination Host Unreachable


---

## Why Linux-Side Fixes Failed

Extensive troubleshooting was attempted inside the guest OS:

- Netplan reconfiguration
- Restarting `systemd-networkd`
- Flushing routes
- Flushing ARP cache
- Forcing DHCPv4
- Disabling cloud-init

None of these resolved the issue.

Once the hypervisor stopped responding to ARP for the VM, **no guest-side command could restore connectivity**.

This marked a clear boundary between:
- Guest OS responsibility
- Hypervisor networking responsibility

---

## Resolution

The Linux VM was rebuilt from scratch with a stable NIC topology:

- External/NAT NIC defined at install time
- No post-install interface changes
- Networking validated before additional configuration

With consistent assumptions across cloud-init, netplan, the kernel, and the hypervisor, connectivity was restored immediately.

---

## Key Takeaway

When troubleshooting Linux networking in virtualized environments, failures may not exist within the guest OS at all.

If Layer 2 adjacency is broken at the hypervisor boundary, Linux can appear correctly configured while being fundamentally isolated.
