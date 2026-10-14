# Case study — DFS Replication initialization warning

**Environment:** Windows Server 2022, VMware Workstation Pro  
**Domain:** `lab.local`  
**Domain controller:** `DC01`  
**Status:** Service and SYSVOL checks looked healthy afterward; final `dcdiag` validation pending

## What happened

While checking my new domain controller after promotion, I ran `dcdiag /q`. It reported that DC01 failed the **DFSREvent** test. I didn't want to assume the entire domain was broken based on one test, but I also didn't want to ignore the warning.

## What I checked

- Confirmed the **DFS Replication** service was running.
- Opened the DFS Replication event log and compared the warnings with later entries.
- Found initial domain connectivity and WMI provider registration errors.
- Saw **Event ID 4602**, indicating SYSVOL initialization had completed.
- Checked the replicated folder and saw **State 4 (Normal)**.
- Verified that **SYSVOL** and **NETLOGON** shares existed and were accessible.

## What I concluded

Later DFSR events showed successful Active Directory configuration updates and service initialization. That suggests the earlier errors were related to initial startup, rather than a failure still affecting SYSVOL.

A later `dcdiag /test:sysvolcheck` passed, which provides additional evidence that SYSVOL is available. I **haven't marked the DFSREvent warning fully resolved** because I still need a clean follow-up `dcdiag /q` or `dcdiag /test:DFSREvent` result. I also haven't verified replication between multiple domain controllers; this is a single-DC lab.

## What I'll do next

```powershell
dcdiag /q
dcdiag /test:sysvolcheck
dcdiag /test:advertising
Get-Service DFSR
Get-SmbShare -Name SYSVOL,NETLOGON
```

I'll record the output and compare it with the event log timestamps. If the DFSREvent warning remains, I'll investigate that result before making any repair changes.

## What I learned

An error in the event log doesn't always mean the same problem is happening right now. Looking at the timeline, the service state, and the current SYSVOL configuration gave me a better picture than reacting to the first warning. I also learned to keep a final validation step open until I've actually run it.
