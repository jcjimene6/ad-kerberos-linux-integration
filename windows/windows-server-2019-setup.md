# Windows Server 2019 Domain Controller Setup

## Overview

This document describes the initial setup of the Windows Server 2019 virtual machine used as the Active Directory Domain Controller (DC) for this lab.

The DC provides:
- Active Directory Domain Services
- DNS
- DHCP
- Routing and NAT (via RRAS)

It serves as the central identity, naming, and routing authority for the Linux client.

---

## Virtual Machine Configuration

- Hypervisor: Oracle VirtualBox
- Operating System: Windows Server 2019
- Network Interfaces:
  - **Internal NIC**: Connected to the internal lab network
  - **External NIC**: Connected to a NAT network for outbound internet access

The dual-NIC configuration allows the DC to function as both:
- An identity/DNS server for internal clients
- A routing gateway for outbound traffic

---

## Active Directory Configuration

After installation, the following were configured:

- Active Directory Domain Services role installed
- New domain created
- Organizational Unit created:
  - `_ADMINS`  
    Used for privileged accounts and delegation testing

DNS was automatically integrated with Active Directory.

---

## DHCP Configuration

The DHCP role was installed and configured with the following scope:

- Subnet: `172.16.0.0/24`
- Address range: `172.16.0.100 – 172.16.0.200`
- Default gateway: Domain Controller
- DNS server: Domain Controller

This allowed internal clients to:
- Receive consistent addressing
- Resolve the AD domain
- Route traffic through the DC

---

## Routing Preparation

The Remote Access role was installed to support routing and NAT functionality.

At this stage:
- RRAS was present
- NAT and IP forwarding were not yet fully enabled

This distinction became important during later troubleshooting and is covered in a separate document.
