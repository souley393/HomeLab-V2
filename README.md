# HomeLab-V2 — Building My Cybersecurity Lab

I'm rebuilding my home lab to get more comfortable working with Windows networks, security logs, and the kind of investigations a SOC analyst handles. Studying for Security+ gave me a foundation, but I wanted to test things myself instead of just reading about them.

The plan is to build a small Active Directory environment in VMware, send Windows logs to a SIEM, and eventually test whether I can spot activity coming from my own Kali VM. This is a work in progress. I'm keeping notes as I go, including problems I run into.

## Current progress

**Phase 1 — VMware networking and Windows Server preparation:** I configured DC01 on the VMnet1 host-only network, gave it a static address, checked its network settings in PowerShell, and saved a snapshot before installing Active Directory. I captured five screenshots of the setup and validation steps. They're linked in the Phase 1 notes.

Read my [Phase 1 notes](docs/01-networking-and-dc01.md).

**Phase 2 — Active Directory:** I created the `lab.local` domain and verified core services. SYSVOL and NETLOGON tests passed, though `dcdiag /q` still reports a DFSREvent warning within its historical event window. I also created and verified six top-level Organizational Units for IT, employees, workstations, member servers, security groups, and service accounts.

[Domain Controller notes](docs/02-active-directory-domain-controller.md) · [OU structure and verification](docs/03-active-directory-ou-structure.md) · [DFSR warning case study](case-studies/dfsr-initialization-warning.md)

## Lab layout

| Machine | Planned address | Purpose |
| --- | --- | --- |
| DC01 — Windows Server 2022 | `192.168.217.10` | Domain controller and DNS |
| Windows client | `192.168.217.20` | Domain-joined endpoint |
| Ubuntu Server | `192.168.217.30` | Planned Wazuh SIEM |
| Kali Linux | `192.168.217.40` | Authorized test machine |

The Windows Server address is shown in my setup evidence. Other addresses are the intended layout and **not yet verified as configured**.

VMnet1 uses `192.168.217.0/24` for host-only connectivity. I'll use NAT only when updates or downloads are needed, rather than bridging the lab to my everyday network.

## What's next

- Finish remaining DC health checks and confirm the DFSREvent warning has cleared.
- Create test user accounts and role-based security groups in the new OU structure.
- Join the Windows client and configure Group Policy and useful audit events.
- Set up Wazuh on Ubuntu and bring in endpoint logs.
- Run controlled tests and document the alerts, evidence, and anything I missed.

I'll mark steps complete when I've verified them, not just when I've attempted them.

## Portfolio

[My cybersecurity portfolio](https://souley393.github.io) · [Phase 1 build notes](docs/01-networking-and-dc01.md)

> **Lab safety:** All testing stays on systems I own and control. No real credentials, hashes, private keys, or enrollment secrets will be committed to this repository.
