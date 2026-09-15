# Incident Report INC-001 - DC01 Samba AD Deployment

## Incident Summary

During the deployment of the DC01 Active Directory Domain Controller,
several configuration and service issues were encountered.

The issues affected Samba services, DNS resolution, network interface
selection, and Kerberos authentication.

All identified issues were resolved and the Domain Controller was
successfully brought to an operational state.

## Incident Information

| Field | Details |
|---|---|
| Incident ID | INC-001 |
| System | DC01 |
| Service | Samba Active Directory |
| Environment | Acme Solutions Ltd. Lab |
| Severity | Medium |
| Status | Resolved |
| Impact | AD deployment and authentication testing were temporarily affected |

## Issue 1 - Samba Service Port Conflict

### Problem

The normal `smbd` service was running and using SMB ports 139/445.
This conflicted with the Samba Active Directory Domain Controller
configuration.

### Investigation

The listening ports were checked using:

    sudo ss -ltnp | grep -E ':139|:445'

This showed that another Samba service was already using the required
ports.

### Resolution

The normal Samba services were stopped and disabled/masked.

The dedicated `samba-ad-dc` service was then used for the Active
Directory environment.

### Verification

    sudo systemctl is-active samba-ad-dc

Result:

`active`

---

## Issue 2 – DNS Resolver Configuration

### Problem

Samba DNS update operations reported a resolver configuration problem.

### Investigation

The server DNS configuration and Samba DNS settings were reviewed.

### Resolution

DC01 was configured to use the local Samba DNS service and the DNS
configuration was corrected.

Samba was also configured to use the intended management interface.

### Verification

    dig @10.10.0.10 dc01.acme.local A

Result:

`10.10.0.10`

---

## Issue 3 - Multiple Network Interfaces

### Problem

DC01 had more than one network interface:

- `ens160` - Management network (`10.10.0.10`)
- `ens256` - VMware network

This could cause Samba services to bind to the wrong interface.

### Resolution

Samba was configured to use the intended management interface:

    interfaces = ens160
    bind interfaces only = yes

### Verification

DNS and Samba services were tested again after the interface
configuration was changed.

---

## Issue 4 - Kerberos KDC Discovery

### Problem

The initial `kinit Administrator` command failed because the
Kerberos client could not correctly locate the KDC.

### Resolution

The Kerberos realm and KDC configuration were corrected so that
`ACME.LOCAL` uses DC01 as the KDC.

### Verification

    kinit Administrator

A Kerberos ticket was successfully obtained.

The ticket was verified using:

    klist

Result:

`Administrator@ACME.LOCAL`

---

## Issue 5 - Administrator Authentication

### Problem

The initial Administrator password was rejected during Kerberos
authentication testing.

### Resolution

The Samba Administrator password was reset using:

    sudo samba-tool user setpassword Administrator

### Verification

Kerberos authentication was tested again:

    kinit Administrator

Authentication succeeded and a valid Kerberos ticket was obtained.

---

## Final Verification

The following checks were completed successfully:

- Samba AD DC service is active.
- DNS resolves `dc01.acme.local`.
- Kerberos SRV records are available.
- Administrator can obtain a Kerberos ticket.
- `testparm` confirms the Active Directory DC role.
- `samba-tool dbcheck --cross-ncs` completed with 0 errors.

## Root Cause / Lessons Learned

The issues were primarily caused by service conflicts, DNS configuration,
multiple network interfaces, and initial Kerberos configuration.

The troubleshooting process demonstrated the importance of checking:

1. Running services
2. Listening ports
3. Network interfaces
4. DNS resolution
5. Kerberos configuration
6. Service health after each change

## Incident Resolution

**Status: RESOLVED**

DC01 is operational and ready for the next Identity & Access Management
configuration tasks.