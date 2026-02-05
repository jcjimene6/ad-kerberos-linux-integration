# Kerberos Client Configuration on Linux

## Overview

This document describes the configuration and validation of Kerberos on the Linux host prior to full Active Directory integration with SSSD.

At this stage of the lab:
- Kerberos authentication was functional
- Ticket issuance and caching worked correctly
- Domain-backed logins were not yet possible

This distinction was critical to diagnosing later failures.

---

## Package Installation

The following packages were installed to support Kerberos authentication and Active Directory integration:
```bash
sudo apt install -y \
  krb5-user \
  sssd \
  sssd-tools \
  libnss-sss \
  libpam-sss \
  adcli \
  realmd \
  oddjob \
  oddjob-mkhomedir \
  packagekit
```

These packages provide:

- Kerberos client utilities such as `kinit` and `klist`
- Active Directory domain discovery and join tooling
- Identity and authentication integration via SSSD

---

## Kerberos Realm Considerations

Kerberos is case-sensitive with respect to realm names.

The following conventions were used:

- DNS domain names are lowercase (for example: `mydomain.com`)
- Kerberos realm names are uppercase (for example: `MYDOMAIN.COM`)

Using incorrect case for the realm name results in authentication failures even when DNS resolution succeeds.

---

## Kerberos Configuration

Kerberos client configuration was defined in the following file:

`/etc/krb5.conf`

Configuration used:

```ini

[libdefaults]
  default_realm = MYDOMAIN.COM
  dns_lookup_realm = true
  dns_lookup_kdc = true
  rdns = false
  forwardable = true
  proxiable = true
  ticket_lifetime = 24h
  renew_lifetime = 7d

[realms]
  MYDOMAIN.COM = {
    kdc = dc.mydomain.com
    admin_server = dc.mydomain.com
  }

[domain_realm]
  .mydomain.com = MYDOMAIN.COM
  mydomain.com = MYDOMAIN.COM
```
DNS-based discovery was intentionally enabled to reduce hardcoding and better reflect a typical Active Directory environment.

---

## Kerberos Validation

Kerberos functionality was validated using the following steps:

kdestroy
sudo systemctl restart systemd-resolved
kinit Administrator@MYDOMAIN.COM
klist

Successful validation confirmed:

- Communication with the Key Distribution Center (KDC)
- Correct Kerberos realm configuration
- Ticket issuance and caching on the Linux host

At this point, Kerberos authentication itself was working as expected.

---

## Partial Success: What Worked and What Did Not

At this stage, Kerberos was operational, but full system integration was not yet complete.

What worked:

- Kerberos tickets could be successfully requested
- Tickets were cached locally on the Linux host
- Authentication succeeded at the Kerberos protocol level

What did not work:

- Domain users could not log into the Linux system
- User and group resolution failed

This demonstrated that Kerberos was functioning correctly, but identity resolution and system login were not yet operational.

Those remaining issues were isolated to SSSD and are covered in a separate document.

