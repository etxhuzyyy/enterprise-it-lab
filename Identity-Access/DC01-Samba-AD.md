# DC01 - Samba Active Directory Domain Controller

## Purpose

This document records the deployment and configuration of the Active Directory
Domain Controller for Acme Solutions Ltd.

Because the lab is running on an Apple Silicon Mac (ARM64) and Windows Server
was not available for this environment, Ubuntu Server ARM64 with Samba Active
Directory Domain Controller was used.

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

A static IP was assigned because the Domain Controller must have a
stable address for DNS, Kerberos, and client authentication.

## Hostname Configuration

The server hostname was configured as:

`dc01.acme.local`

The local hosts file was updated so that the hostname could resolve to the
static IP address.

## Samba AD Deployment

Samba Active Directory was installed and configured as the domain controller.

The domain was provisioned using:

    sudo samba-tool domain provision \
      --domain ACME \
      --realm=ACME.LOCAL \
      --server-role=dc \
      --use-rfc2307 \
      --dns-backend=SAMBA_INTERNAL

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

    dig @10.10.0.10 dc01.acme.local A

Result:

`10.10.0.10`

Kerberos service records were also verified:

    dig @10.10.0.10 _kerberos._tcp.acme.local SRV

## Kerberos Authentication

Kerberos was configured for the `ACME.LOCAL` realm.

Authentication was tested using:

    kinit Administrator

The Kerberos ticket was verified with:

    klist

A valid ticket for:

`Administrator@ACME.LOCAL`

was successfully obtained.

## Service Configuration

The normal Samba file-server services were disabled so that the
Active Directory Domain Controller service could manage the required
AD services.

The Samba AD DC service was enabled and verified:

    sudo systemctl is-active samba-ad-dc

Result:

`active`

## Problems Encountered

### SMB Port Conflict

The `smbd` service was initially using ports 139/445, which conflicted
with the Samba AD DC configuration.

**Resolution:** The normal Samba services were stopped and masked, and
` samba-ad-dc` was used as the primary service.

### DNS Resolver Issue

Samba initially reported a resolver configuration problem.

**Resolution:** The server's DNS configuration was changed to use the
local Samba DNS service, and Samba was configured to use the intended
network interface.

### Multiple Network Interfaces

DC01 had both the management interface and a VMware network interface.

**Resolution:** Samba was configured to use the intended management
interface (`ens160`) for the AD network.

### Kerberos Authentication

Initial Kerberos authentication failed because the KDC could not be
properly discovered and the Administrator password required resetting.

**Resolution:** The Kerberos realm/KDC configuration was corrected and
the Administrator password was reset.

## Verification

The following checks were completed successfully:

- Samba AD DC service is active.
- `testparm` confirms the server role is an Active Directory DC.
- DNS resolves `dc01.acme.local` to `10.10.0.10`.
- Kerberos SRV records are available.
- `kinit Administrator` successfully obtains a Kerberos ticket.
- `klist` confirms the Kerberos ticket.
- `samba-tool dbcheck --cross-ncs` completed with **0 errors**.

## Final Status

**DC01 Active Directory foundation: Operational**

The Samba-based Active Directory environment is ready for the next
Identity & Access Management tasks, including creating organizational
units, users, groups, and applying access controls.