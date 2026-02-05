# SSSD Runtime Failure and Recovery

## Overview

This document describes the failure modes encountered while configuring SSSD on the Linux host and the steps taken to restore stable domain-backed authentication.

At this stage of the lab:
- Kerberos authentication was functioning correctly
- Active Directory domain join had partially succeeded
- Domain users could not log in due to SSSD failures

The root cause was not configuration syntax, but SSSD runtime state.

---

## Symptoms

SSSD exhibited multiple failure behaviors:

- The service crashed immediately on startup
- Required database files were reported as missing
- Failures occurred without clear error messages
- Repeated core dumps were generated
- Configuration validation succeeded, but runtime initialization failed

Despite correct Kerberos behavior, identity resolution and PAM/NSS integration did not function.

---

## Why This Was Difficult to Diagnose

SSSD is highly sensitive to its runtime environment.

It requires:
- Strict file and directory permissions
- A specific directory structure
- Valid backend state for IPC, caching, and databases

A single missing directory or incorrect permission is sufficient to cause complete failure, often without a clear error at the configuration level.

---

## Initial Verification

Before rebuilding runtime state, the following checks were performed:

- Verified `sssd.conf` syntax
- Confirmed configuration file permissions (`600`, owned by `root`)
- Ensured the configuration passed validation using `sssctl config-check`

These checks confirmed that the configuration itself was not the cause of failure.

---

## Root Cause

During earlier troubleshooting, the private IPC socket directory used by SSSD had been accidentally deleted.

Specifically, the following path was missing:

/var/lib/sss/pipes/private

Without this directory, SSSD could not establish internal communication channels and failed silently during startup.

---

## Recovery Procedure

SSSD runtime state was rebuilt manually to restore required directories and permissions.

### Stop SSSD
```bash
sudo systemctl stop sssd
```
### Recreate Required Directories

```bash
sudo mkdir -p \
  /var/lib/sss/pipes/private \
  /var/lib/sss/db \
  /var/lib/sss/mc \
  /var/log/sssd
```
### Correct Ownership and Permissions

```bash
sudo chown -R root:root /var/lib/sss /var/log/sssd

sudo chmod 755 /var/lib/sss /var/lib/sss/pipes
sudo chmod 700 /var/lib/sss/pipes/private /var/lib/sss/db /var/lib/sss/mc
sudo chmod 700 /var/log/sssd
```

### Remove Stale Sockets
```bash
sudo rm -f /var/lib/sss/pipes/* /var/lib/sss/pipes/private/*
```
### Verify Configuration Security 
```bash
sudo chown root:root /etc/sssd/sssd.conf
sudo chmod 600 /etc/sssd/sssd.conf
sudo sssctl config-check
```
### Restart SSSD
```bash
sudo systemctl start sssd
sudo systemctl status sssd --no-pager -l
```
