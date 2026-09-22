# DC01 - Samba Active Directory Domain Controller

## Purpose

This document records the deployment and configuration of the Active Directory Domain Controller for Acme Solutions Ltd.

Because the lab is running on an Apple Silicon Mac (ARM64) and Windows Server was not available for this environment, Ubuntu Server ARM64 with Samba Active Directory Domain Controller was used.

## Server Configuration

| Setting | Value |
|---|---|
| Hostname | DC01 |
| FQDN | dc01.acme.local |
| Domain | acme.local |
| NetBIOS Domain | ACME |
| Server IP | 10.10.0.10/24 |
| AD Role | Active Directory Domain Controller |
| DNS | Samba Internal DNS |
| OS | Ubuntu Server ARM64 |

## Network Configuration

DC01 was connected to the management network:

- Network: `10.10.0.0/24`
- DC01: `10.10.0.10`
- Domain: `acme.local`

A static IP was assigned because the Domain Controller requires a stable address for DNS, Kerberos, and client authentication.

## Hostname Configuration

The server hostname was configured as:

`dc01.acme.local`

The local hosts file was updated so that the hostname could resolve to the static IP address.

## Samba AD Deployment

Samba Active Directory was installed and configured as the domain controller.

The domain was provisioned using:

```bash
sudo samba-tool domain provision \
  --domain ACME \
  --realm=ACME.LOCAL \
  --server-role=dc \
  --use-rfc2307 \
  --dns-backend=SAMBA_INTERNAL
```

The resulting configuration uses:

- Domain: `ACME`
- Kerberos Realm: `ACME.LOCAL`
- DNS Domain: `acme.local`
- Role: Active Directory Domain Controller
- DNS Backend: Samba Internal DNS

## DNS Configuration

Active Directory depends heavily on DNS for service discovery.

DC01 was configured to use its own Samba DNS service:

`127.0.0.1`

DNS was tested with:

```bash
dig @10.10.0.10 dc01.acme.local A
```

Result:

`10.10.0.10`

Kerberos service records were also verified:

```bash
dig @10.10.0.10 _kerberos._tcp.acme.local SRV
```

## Kerberos Authentication

Kerberos was configured for the `ACME.LOCAL` realm.

Authentication was tested using:

```bash
kinit Administrator
```

The Kerberos ticket was verified with:

```bash
klist
```

A valid ticket for `Administrator@ACME.LOCAL` was successfully obtained.

## Service Configuration

The normal Samba file-server services were disabled so that the Active Directory Domain Controller service could manage the required AD services.

The Samba AD DC service was enabled and verified:

```bash
sudo systemctl is-active samba-ad-dc
```

Result:

`active`

## Identity and Access Management

### Organizational Units

The following OU structure was created:

```text
ACME.LOCAL
├── Users
│   ├── Developers
│   ├── IT
│   ├── Management
│   └── HR
├── Groups
├── Computers
└── Domain Controllers
```

### Security Groups

The following security groups were created:

| Group | Purpose |
|---|---|
| GG-Developers | Access for development staff |
| GG-IT | Access for IT staff |
| GG-Management | Access for management staff |
| GG-HR | Access for HR staff |

### User Accounts

The following test users were created and assigned to their departmental OUs and security groups:

| User | Department | Security Group |
|---|---|---|
| alice.johnson | Developers | GG-Developers |
| bob.smith | IT | GG-IT |
| carol.williams | Management | GG-Management |
| david.brown | HR | GG-HR |

### Authentication Testing

Kerberos authentication was tested successfully using domain user accounts.

A valid Kerberos ticket for `alice.johnson@ACME.LOCAL` was successfully obtained.

## Windows Domain Client

A Windows 11 Pro ARM virtual machine was configured as `CLIENT01`.

CLIENT01 was configured to use DC01 (`10.10.0.10`) for DNS and successfully joined the `acme.local` domain.

The computer account was created in Active Directory as:

`CLIENT01$`

## Domain Login Verification

Domain authentication was successfully tested on CLIENT01 using:

`ACME\alice.johnson`

The following checks were successful:

- `whoami` confirmed the domain user identity.
- Windows reported that CLIENT01 is part of the domain.
- `nltest /dsgetdc:acme.local` successfully located DC01.
- `whoami /groups` confirmed the user's departmental security group membership.

## Problems Encountered

### SMB Port Conflict

The `smbd` service was initially using ports 139/445, which conflicted with the Samba AD DC configuration.

**Resolution:** The normal Samba services were stopped and masked, and `samba-ad-dc` was used as the primary service.

### DNS Resolver Issue

Samba initially reported a resolver configuration problem.

**Resolution:** The server's DNS configuration was changed to use the local Samba DNS service, and Samba was configured to use the intended network interface.

### Multiple Network Interfaces

DC01 had both the management interface and a VMware network interface.

**Resolution:** Samba was configured to use the intended management interface (`ens160`) for the AD network.

### Kerberos Authentication

Initial Kerberos authentication failed because the KDC could not be properly discovered and the Administrator password required resetting.

**Resolution:** The Kerberos realm/KDC configuration was corrected and the Administrator password was reset.

### CLIENT01 Domain Join Failure

CLIENT01 initially failed to join the `acme.local` domain and Windows displayed a misleading storage-related error.

Investigation of `C:\Windows\debug\NetSetup.log` and the CLIENT01 system showed that the Windows C: drive had `0 bytes` free.

**Resolution:** Free disk space was restored on CLIENT01. The domain join was then attempted again and completed successfully.

## Verification

The following checks were completed successfully:

- Samba AD DC service is active.
- `testparm` confirms the server role is an Active Directory DC.
- DNS resolves `dc01.acme.local` to `10.10.0.10`.
- Kerberos SRV records are available.
- `kinit Administrator` successfully obtains a Kerberos ticket.
- `klist` confirms the Kerberos ticket.
- `samba-tool dbcheck --cross-ncs` completed with **0 errors**.
- AD organizational units were created and verified.
- Security groups were created and verified.
- Domain users were created, assigned to OUs, and added to groups.
- CLIENT01 successfully joined the `acme.local` domain.
- Domain user authentication and group membership were verified on CLIENT01.

## Final Status

**DC01 Active Directory foundation: Operational**

The Samba-based Active Directory environment is operational. The environment now supports centralized identity management, Kerberos authentication, security groups, organizational units, and Windows domain client authentication.

The environment is ready for the next server infrastructure and endpoint management tasks.
