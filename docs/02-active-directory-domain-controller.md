# Phase 2 — Building the Active Directory domain

**Environment:** Windows Server 2022 on VMware Workstation Pro  
**Domain controller:** `DC01` (`192.168.217.10`)  
**Domain / forest:** `lab.local`  
**Status:** AD DS promotion and core checks completed; final health checks still in progress

## What I did

After setting up the network in Phase 1, I installed the **Active Directory Domain Services** role on DC01. The role installation wizard finished successfully and prompted me to promote the server. I then created a new forest named `lab.local`.

I didn't keep every page of the promotion wizard, so I'm using the commands and results from the running server to show that the domain was created. The screenshots I have include the AD DS role installation result, `Get-ADDomain`, `Get-ADDomainController`, DNS records, services, and logon shares.

## What I checked after promotion

| Check | What I observed |
| --- | --- |
| `Get-ADDomain` | Returned the `lab.local` domain and `DC01.lab.local` as the PDC emulator |
| `Get-ADForest` | Returned forest `lab.local`, with DC01 as the global catalog |
| `Get-ADDomainController` | Reported DC01 at `192.168.217.10`, enabled and a global catalog |
| `Get-Service NTDS,DNS,KDC,Netlogon,W32Time` | All five services running, startup type Automatic |
| DNS A record | `dc01.lab.local` resolved to `192.168.217.10` |
| LDAP SRV record | `_ldap._tcp.dc._msdcs.lab.local` pointed to DC01 on port 389 |
| Kerberos SRV record | `_kerberos._tcp.lab.local` pointed to DC01 on port 88 |
| `nltest /dsgetdc:lab.local` | Found DC01 successfully |
| `klist` | Showed a Kerberos TGT for the Administrator session |
| `net share` and `Test-Path` | SYSVOL and NETLOGON shares existed and were accessible |

Some commands I used:

```powershell
Get-ADDomain
Get-ADForest
Get-ADDomainController
Get-Service NTDS,DNS,KDC,Netlogon,W32Time | Select-Object Name,Status,StartType
Resolve-DnsName dc01.lab.local
Resolve-DnsName -Type SRV _ldap._tcp.dc._msdcs.lab.local
Resolve-DnsName -Type SRV _kerberos._tcp.lab.local
nltest /dsgetdc:lab.local
klist
net share
Test-Path "\\dc01.lab.local\SYSVOL"
Test-Path "\\dc01.lab.local\NETLOGON"
```

One detail I noticed: the DNS client setting on DC01 now shows `127.0.0.1`. That's the local DNS server loopback address, and the name-resolution checks worked. When I join the Windows client, I'll point **that client** to `192.168.217.10` instead.

## A warning I investigated

During validation, `dcdiag /q` reported a **DFSREvent** failure. I checked the DFS Replication service, its event log, the SYSVOL state, and the shares instead of changing settings immediately. The service later reported normal SYSVOL initialization, but a final clean `dcdiag` run is still pending.

I wrote up the investigation separately: [Case study — DFS Replication initialization warning](../case-studies/dfsr-initialization-warning.md).

## Still to verify

- Re-run `dcdiag /q` and save the current output, especially the DFSREvent test.
- **Completed:** `dcdiag /test:dns /v` passed, including DNS authentication, basic, forwarding/root hints, delegation, dynamic updates and record registration.
- **Completed:** `dcdiag /test:sysvolcheck` reported `DC01 passed test SysVolCheck`.
- Still to run or capture: `dcdiag /test:advertising` and a fresh `dcdiag /q` to determine whether the earlier DFSREvent warning has cleared.
- Check the Windows Time configuration. `w32tm /query /status` showed `Local CMOS Clock` as the source, so I need a reliable time plan before domain-joining other machines.
- Publish the saved AD DS installation and verification screenshots after final review. The wizard screenshots for new-forest options and prerequisite checks were not captured.

## Next: Phase 2, Step 4 — Active Directory structure

I'll create organizational units for users, workstations, servers, and groups, then add test accounts and security groups. After that, I'll document the Windows client domain join and Group Policy setup. I want to be able to explain why I organized the directory this way, not just show that I clicked through the menus.
