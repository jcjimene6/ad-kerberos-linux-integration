# Linux Server Initial Setup

## Overview

This document describes the initial setup of the Ubuntu Server virtual machine that was later joined to the Active Directory domain.

The Linux host serves as:
- A Kerberos client
- An AD domain member
- A test system for domain-backed authentication using SSSD

---

## Virtual Machine Configuration

- Operating System: Ubuntu Server 22.04 LTS (Jammy)
- Kernel: 5.15
- Hypervisor: Oracle VirtualBox

During the first installation attempt, the VM was configured with:
- A single **internal NIC** only

At this stage, no external/NAT interface was present.

---

## Installation Notes

After installing Ubuntu Server:
- The installer ISO had to be manually unmounted
- Failure to do so initially caused the VM to repeatedly boot back into the installer

Once corrected, the system booted normally into the installed OS.

---

## Initial Networking Assumptions

At the time of installation:
- The Linux host had no default route to the internet
- The internal NIC had no DHCP-provided gateway
- External connectivity was not yet required

These assumptions were later invalidated when external connectivity was introduced, which led to subtle and difficult-to-diagnose failures related to cloud-init and netplan behavior.

Those failures are documented in a separate troubleshooting section.

---

## Next Steps

After the base installation:
- Static networking was configured
- DNS was pointed at the domain controller
- Kerberos and AD integration packages were installed

Subsequent configuration and troubleshooting are covered in the following documents.
