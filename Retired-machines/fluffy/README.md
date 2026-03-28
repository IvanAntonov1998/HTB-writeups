# Fluffy

**Platform:** Hack The Box  
**Status:** Retired  
**OS:** Windows  
**Category:** Active Directory  
**Focus Areas:** SMB, LDAP, Kerberos, BloodHound, AD CS, Privilege Escalation

## Summary

Fluffy is a Windows Active Directory machine that rewards structured domain enumeration and careful privilege-path analysis. The attack path began with identifying a domain controller exposing standard AD services and eventually led to privilege escalation through certificate-related abuse in the domain environment.

This machine was a good reminder that not every Windows box is about direct exploitation. In this case, success came from understanding the relationships between accounts, groups, and certificate infrastructure rather than from a single obvious weakness.

## Enumeration

Initial reconnaissance showed that the target was a domain controller exposing several typical Active Directory services, including DNS, Kerberos, LDAP, SMB, and WinRM. SMB signing was enabled and required, which immediately helped narrow the range of viable attack paths and pushed the investigation toward authenticated enumeration and privilege analysis.

At this stage, the most important goal was to confirm the domain context, identify available services, and understand how the environment was structured before attempting deeper access.

### Notable findings

- Domain controller identified early during service enumeration
- Standard AD-related ports exposed, including LDAP, Kerberos, SMB, and WinRM
- SMB signing enabled and required
- Certificate infrastructure present in the environment

## Initial Access

After the initial service discovery phase, progress depended on validating access and expanding visibility inside the domain. Enumeration of available resources and relationships revealed a path toward credentialed access and further domain mapping.

Rather than relying only on manual checks, graph-based enumeration became especially valuable here. Tools such as BloodHound / RustHound helped expose connections between users, groups, and privileges that were not immediately obvious from raw service output alone.

## Privilege Escalation

The privilege escalation path centered on Active Directory permissions and certificate-related abuse. Once the relevant relationships and access rights were understood, it became possible to pivot into more privileged accounts and continue escalating through the AD CS attack surface.

What made this box interesting was that the escalation depended less on noisy exploitation and more on correctly interpreting domain permissions, account control, and certificate behavior.

## Why this box was interesting

Fluffy stands out because it combines several Windows concepts into one attack path:

- Classic Active Directory enumeration
- Relationship mapping with BloodHound-style tooling
- Service and account privilege analysis
- AD CS as a privilege escalation vector

This made it a strong exercise in methodology rather than just tool usage.

## Lessons Learned

- Time synchronization matters in Kerberos-heavy environments and should be checked early
- SMB signing requirements can quickly eliminate some assumptions and save time
- BloodHound-style analysis is extremely useful when the path is permission-based rather than exploit-based
- AD CS misconfigurations can create powerful escalation opportunities
- Clean note-taking matters, but public writeups should always be sanitized before publishing

## Tools Used

- Nmap
- smbclient
- NetExec
- BloodHound / RustHound
- Certipy
- Evil-WinRM

## Key Takeaways

This machine reinforced the importance of slowing down and understanding the domain instead of rushing toward exploitation. The full path only became clear after combining service enumeration, account analysis, and certificate-related privilege mapping.

Fluffy is a good example of how Windows privilege escalation often comes from chaining small pieces of domain knowledge together rather than finding one dramatic vulnerability.
