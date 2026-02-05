# Windows Server RRAS Routing and NAT Troubleshooting

## Overview

This document describes the routing and NAT configuration issues encountered on the Windows Server 2019 domain controller and how they affected connectivity for the Linux client.

Although the Remote Access role was installed, the domain controller was not initially functioning as a router or NAT gateway, which caused outbound connectivity failures once Linux DNS was pointed at the DC.

---

## Initial State

At the start of troubleshooting:

- The Remote Access (RRAS) role was installed
- The server had two NICs:
  - Internal NIC connected to the lab network
  - External NIC connected to a NAT network
- The Linux host was configured to use the DC as its DNS server

Despite this setup, Linux clients could not reach external networks.

---

## Key Discovery

Installing the RRAS role alone does **not** automatically enable routing or NAT.

Although RRAS appeared present in Server Manager, the system was not forwarding packets between interfaces.

This meant:
- DNS queries reached the DC
- Kerberos traffic reached the DC
- Outbound traffic destined for the internet stopped at the DC

From the Linux client’s perspective, this appeared as a generic “no route to internet” failure.

---

## Verifying IP Forwarding

To confirm whether the domain controller was acting as a router, IP forwarding was checked at the TCP/IP stack level using PowerShell:

reg query "HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters" /v IPEnableRouter

This registry value controls whether Windows forwards packets between interfaces.

The value was not enabled, confirming that the server was not routing traffic.

---

## Why This Caused Linux Failures

Once the Linux host’s DNS was pointed at the domain controller, the DC became a critical dependency for all outbound communication.

At that point:

- DNS queries were successfully reaching the DC
- Kerberos traffic was successfully reaching the DC
- The Linux host treated the DC as its default gateway

However, because IP forwarding and NAT were not enabled on the domain controller, the DC silently dropped packets that were not destined for itself.

From the Linux client’s perspective, this created a misleading failure mode:

- DNS resolution worked
- Internal connectivity worked
- Kerberos partially worked
- Internet access failed completely

This made the issue appear to be a Linux-side routing or DNS problem, even though the failure existed entirely on the Windows side.

---

## Resolution

Routing and NAT were explicitly enabled on the Windows Server domain controller.

This included:
- Enabling IP forwarding at the TCP/IP stack level
- Configuring RRAS to perform NAT between the internal and external NICs

Once packet forwarding was active:

- Linux traffic successfully traversed the DC
- NAT translated outbound traffic correctly
- External connectivity was restored immediately

No configuration changes were required on the Linux host after routing was corrected.

---

## Validation

Successful resolution was verified by performing the following checks:

- Confirmed outbound connectivity from the Linux host
- Successfully ran package updates
- Observed consistent routing behavior through the domain controller

These checks confirmed that the domain controller was functioning as a proper gateway for downstream clients.

---

## Key Takeaway

When a Windows Server is expected to act as a router, installing the RRAS role alone is not sufficient.

IP forwarding and NAT must be explicitly enabled. Otherwise, downstream systems may appear correctly configured while losing outbound connectivity silently at the gateway.

