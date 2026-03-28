#  Footprinting Lab - Medium

- **Platform:** HTB Academy / Lab
- **Status:** Retired / Practice Lab
- **OS:** Windows Server 2019
- **Category:** Footprinting / Enumeration
- **Focus Areas:** Nmap, SMB, NFS, RDP, WinRM, Shares, Config Exposure, MSSQL, Credential Reuse

## Summary

This lab focused less on exploitation and more on disciplined enumeration. The target exposed a mixed Windows environment with SMB, RPC, RDP, WinRM, and NFS-related services. The key to solving it was not finding a single dramatic vulnerability, but correlating information from multiple protocols and following the trail from exposed shares to application configuration, then to internal access, and finally to the credential needed for the target user.

What made this lab valuable was the way it rewarded careful service analysis. Each step by itself looked ordinary, but together they revealed the full attack path.

## Enumeration

Initial scanning identified a Windows Server 2019 host exposing several services of interest, including SMB, RPC, RDP, WinRM, and NFS-related functionality. SMB signing was enabled but not required, and the target appeared to be operating in a workgroup-style context rather than as a normal Active Directory member.

From a footprinting perspective, the most important early observations were:

- Mixed Windows and NFS exposure on the same target
- SMB shares available with valid credentials
- RDP exposed and reachable
- WinRM exposed but not immediately useful from every network path
- NFS export available to everyone, which became a major source of information later on

### Notable ports and services

- `111/tcp` - rpcbind
- `135/tcp` - MSRPC
- `139/tcp` / `445/tcp` - SMB / NetBIOS
- `2049/tcp` - NFS-related service
- `3389/tcp` - RDP
- `5985/tcp` - WinRM
- `47001/tcp` - Microsoft HTTP API endpoint

## SMB and Initial Access

Using the credentials recovered during enumeration, SMB access succeeded and revealed several shares, including administrative shares, a development-oriented share, and user-accessible content. This confirmed that the credentials were valid and worth testing across other services as well.

This was a good reminder that once working credentials are found, they should be validated broadly but carefully. In this lab, SMB access was not the final objective, but it confirmed the identity and provided confidence that password reuse might exist elsewhere in the environment.

## NFS Enumeration

One of the most useful discoveries came from the NFS side. Enumeration showed that a support-related export was accessible broadly, and mounting it exposed a large number of ticket files. Most of them were not useful, but one contained a conversation that leaked application and mail configuration details.

That ticket became the turning point of the lab. It exposed:

- A service hostname
- A username
- A password
- Email configuration context
- Indicators of a web application environment
- A reference to an internal address and port associated with the application stack

This was a classic example of why support data and configuration artifacts matter so much in real assessments.

## Credential Reuse and Service Validation

After the leaked credentials were identified, the next step was to validate them across the available services. Some access paths did not work from the attacker’s position, which suggested segmentation or route constraints, but the same credentials worked successfully over RDP to the main host.

This was another good lesson from the lab: failed access does not always mean bad credentials. Sometimes the issue is context, network reachability, or the service endpoint being different from what the leaked configuration initially suggests.

## Host Enumeration

Once on the system, local enumeration confirmed the host identity, Windows version, basic group memberships, and the presence of SQL Server Management Studio. The account had standard user-style access with remote desktop rights, but not obvious administrative privileges.

At this point, the solution depended on shifting from service enumeration to workstation-style triage:

- Inspecting desktop applications
- Reviewing user-accessible files and shares
- Checking saved artifacts and logs
- Looking for operational shortcuts or reused secrets
- Understanding which systems and resources the logged-in user interacted with regularly 

## MSSQL Discovery

Further exploration revealed development-related material containing SQL credentials. Those credentials enabled access to SQL Server, where application data could be queried directly. From there, it was possible to locate the record for the target user and recover the required proof credentials. 

This was the final chain that completed the lab:

1. Multi-service enumeration
2. Ticket/config discovery through NFS
3. Credential validation and reuse
4. RDP access to the host
5. Discovery of database-related artifacts
6. SQL access and retrieval of the target account data 

## Why this lab was interesting

This lab stands out because it feels realistic. Nothing here depended on a flashy exploit. Instead, it relied on the kind of work that often matters most in real environments:

- Careful network and service enumeration
- Paying attention to support data and mismanaged files
- Reusing validated credentials intelligently
- Thinking across protocols instead of staying locked into one service
- Using database access for information recovery rather than privilege escalation for its own sake

## Lessons Learned

- NFS shares are easy to overlook in Windows-heavy labs, but they can expose high-value operational data
- Support tickets and configuration snippets can reveal more than direct service enumeration
- Always test valid credentials across all relevant services
- When remote management paths fail, consider segmentation before assuming the creds are wrong
- Local applications like SSMS can be strong indicators of the next step
- Enumeration often matters more than exploitation in mixed enterprise-style environments

## Tools Used

- Nmap
- smbclient
- NetExec
- enum4linux
- showmount / NFS client utilities
- RDP client
- WinRM client
- SQL Server Management Studio

## Key Takeaways

Footprinting Lab - Medium is a strong exercise in disciplined enumeration. The solution came from linking together details from multiple services rather than forcing a single path. It is a great reminder that support data, shared storage, and credential reuse can be just as important as the host itself.

The biggest lesson here is simple: good footprinting is often what wins the box.

## Artifacts Observed

- Windows Server 2019 host with SMB, RPC, RDP, WinRM, and NFS exposure
- Accessible NFS export containing support ticket data
- Leaked application and SMTP-related configuration
- Working credential reuse across services
- Development-related share content leading to SQL access
- Recovery of target user credentials from database content
