# Linux <> Active Directory Integration (Kerberos + SSSD)

=======
## Overview
This project documents the process of integrating an Ubuntu 22.04 Linux host with a Windows Server 2019 Active Directory domain using **Kerberos and SSSD**, without winbind or any GUI tools.

The goal was to achieve stable, domain-backed authentication on Linux while understanding — and troubleshooting — the full identity and networking stack end to end.

## Environment
- **Domain Controller:** Windows Server 2019  
  - Active Directory Domain Services  
  - DNS  
  - DHCP  
  - RRAS (Routing / NAT)
- **Linux Client:** Ubuntu Server 22.04 LTS (kernel 5.15)
- **Hypervisor:** Oracle VirtualBox

## Architecture
- The domain controller provides:
  - DNS for the AD domain
  - DHCP for internal clients
  - Routing/NAT for outbound internet access
- The Linux host:
  - Uses the DC for DNS and Kerberos
  - Authenticates domain users via SSSD
  - Obtains Kerberos tickets directly from the KDC

## What Was Configured
- Dual-NIC routing and NAT on the domain controller
- Static networking on Linux using netplan
- DNS and time synchronization prerequisites for Kerberos
- Kerberos client configuration (`/etc/krb5.conf`)
- Active Directory domain join via `realmd` / `adcli`
- SSSD for NSS and PAM integration
- SSH authentication using domain accounts with Kerberos tickets

## Key Challenges & Lessons Learned

### Networking Boundaries Matter
Several failures were caused by mismatches between:
- Linux’s perceived network state
- VirtualBox’s NAT behavior
- The domain controller’s routing configuration

Once the hypervisor stopped answering ARP for a VM, no guest OS command could fix it.

### cloud-init & netplan Interactions
Ubuntu Server records early assumptions about network interfaces during installation.  
Changing NIC topology later can result in configurations that appear correct but are fundamentally broken.

### Kerberos ≠ Identity Resolution
Kerberos ticket issuance can succeed even when:
- SSSD is misconfigured
- NSS/PAM integration is broken
- Domain users cannot log in

Understanding this distinction was critical to effective troubleshooting.

### SSSD Is Stateful and Fragile
SSSD depends on:
- Exact directory structure
- Strict ownership and permissions
- Valid runtime state

A missing private IPC socket directory caused repeated silent failures until the backend state was rebuilt.

## Validation
Successful integration was validated by:
- Obtaining and listing Kerberos tickets (`kinit`, `klist`)
- Resolving domain users and groups (`id`, `getent`)
- Authenticating over SSH using a domain account
- Confirming stable SSSD operation

## Screenshots
Detailed screenshots of configuration steps and troubleshooting states are included in:
