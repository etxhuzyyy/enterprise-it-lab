# Incident Report INC-001 - DC01 Samba AD Deployment

## Incident Summary

During the deployment of the DC01 Active Directory Domain Controller, several configuration and service issues were encountered.

The issues affected Samba services, DNS resolution, network interface selection, and Kerberos authentication. A later domain-join issue involving CLIENT01 was also identified and resolved.

All identified issues were resolved and the Domain Controller was successfully brought to an operational state.

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

This conflicted with the Samba Active Directory Domain Controller configuration.

### Investigation

The listening ports were checked using:

```bash
sudo ss -ltnp | grep -E ':139|:445'
```

This showed that another Samba service was already using the required ports.

### Resolution

The normal Samba services were stopped and disabled/masked.

The dedicated `samba-ad-dc` service was then used for the Active Directory environment.

### Verification

```bash
sudo systemctl is-active samba-ad-dc
```

Result:

`active`

---

## Issue 2 - DNS Resolver Configuration

### Problem

Samba DNS update operations reported a resolver configuration problem.

### Investigation

The server DNS configuration and Samba DNS settings were reviewed.

### Resolution

DC01 was configured to use the local Samba DNS service and the DNS configuration was corrected.

Samba was also configured to use the intended management interface.

### Verification

```bash
dig @10.10.0.10 dc01.acme.local A
```

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

```ini
interfaces = ens160
bind interfaces only = yes
```

### Verification

DNS and Samba services were tested again after the interface configuration was changed.

---

## Issue 4 - Kerberos KDC Discovery

### Problem

The initial `kinit Administrator` command failed because the Kerberos client could not correctly locate the KDC.

### Resolution

The Kerberos realm and KDC configuration were corrected so that `ACME.LOCAL` uses DC01 as the KDC.

### Verification

```bash
kinit Administrator
```

A Kerberos ticket was successfully obtained.

The ticket was verified using:

```bash
klist
```

Result:

`Administrator@ACME.LOCAL`

---

## Issue 5 - Administrator Authentication

### Problem

The initial Administrator password was rejected during Kerberos authentication testing.

### Resolution

The Samba Administrator password was reset using:

```bash
sudo samba-tool user setpassword Administrator
```

### Verification

Kerberos authentication was tested again:

```bash
kinit Administrator
```

Authentication succeeded and a valid Kerberos ticket was obtained.

---

## Issue 6 - CLIENT01 Domain Join Failure

### Problem

CLIENT01 initially failed to join the `acme.local` domain.

Windows displayed a misleading storage-related error indicating that there was not enough space.

### Investigation

The Windows domain-join log was reviewed:

```text
C:\Windows\debug\NetSetup.log
```

The CLIENT01 computer account was found to exist in Samba AD, indicating that the join process had progressed to computer-account creation.

Further investigation showed that the CLIENT01 C: drive had:

`0 bytes free`

### Root Cause

The Windows CLIENT01 virtual machine had completely exhausted its available C: drive space.

### Resolution

Free disk space was restored on CLIENT01.

The domain join was then attempted again and completed successfully.

Windows displayed:

`Welcome to the acme.local domain`

CLIENT01 was restarted after the successful domain join.

### Verification

The following checks were completed successfully on CLIENT01:

- `whoami` confirmed the domain user identity.
- Windows confirmed CLIENT01 was part of the `acme.local` domain.
- `nltest /dsgetdc:acme.local` successfully located DC01.
- `whoami /groups` confirmed the user's departmental security group membership.

---

## Final Verification

The following checks were completed successfully:

- Samba AD DC service is active.
- DNS resolves `dc01.acme.local`.
- Kerberos SRV records are available.
- Administrator can obtain a Kerberos ticket.
- `testparm` confirms the Active Directory DC role.
- `samba-tool dbcheck --cross-ncs` completed with **0 errors**.
- CLIENT01 successfully joined the `acme.local` domain.
- Domain user authentication and group membership were verified on CLIENT01.

## Root Cause / Lessons Learned

The initial DC01 issues were primarily caused by service conflicts, DNS configuration, multiple network interfaces, and initial Kerberos configuration.

The CLIENT01 domain-join issue was caused by the Windows VM having no free space on its C: drive.

The troubleshooting process demonstrated the importance of checking:

1. Running services
2. Listening ports
3. Network interfaces
4. DNS resolution
5. Kerberos configuration
6. Disk space on client systems
7. Service health after each change
8. Domain-join logs when Windows reports a generic error

## Incident Resolution

**Status: RESOLVED**

DC01 is operational, the `acme.local` Active Directory environment is functioning, and CLIENT01 is successfully joined to the domain.
