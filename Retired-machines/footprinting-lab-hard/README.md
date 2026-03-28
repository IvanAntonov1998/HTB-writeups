# Footprinting Lab - Hard

- **Platform:** HTB Academy / Lab
- **Status:** Retired / Practice Lab
- **OS:** Linux
- **Category:** Footprinting / Enumeration
- **Focus Areas:** Nmap, SNMP, POP3, IMAP, SSH, Credential Reuse, Local Enumeration, MySQL

## Summary

Footprinting Lab - Hard is a Linux-based lab that emphasizes patient service enumeration and cross-protocol correlation rather than flashy exploitation. The target exposed mail services, SSH, and SNMP, and the path to success depended on extracting operational information from those services, validating credentials carefully, and following the trail until the target account credentials were recovered.

What made this lab valuable was that each service contributed a different part of the picture. Mail access revealed one kind of artifact, SNMP revealed another, and local enumeration after SSH access completed the path.

## Enumeration

The initial scan showed a Linux host exposing SSH, POP3, IMAP, and SNMP. Both encrypted and unencrypted mail services were available, immediately suggesting that mailbox access and mail-related artifacts might be central to the lab. SNMP also stood out as an important avenue because it can expose operational details that are not visible through standard TCP service enumeration.

### Notable services

- `22/tcp` - OpenSSH
- `110/tcp` - POP3
- `143/tcp` - IMAP
- `993/tcp` - IMAPS
- `995/tcp` - POP3S
- `161/udp` - SNMP 

The target was identified as `NIXHARD`, and the mail stack appeared to be based on Dovecot running on Ubuntu. That already suggested a likely path involving mailbox access, stored credentials, or internal operational communications.
## Mail Service Analysis

POP3 and IMAP became a major focus early on. Manual interaction with the services confirmed their capabilities and helped validate that the host was behaving like a real internal mail system rather than a decoy service. Once working credentials were identified, mailbox access exposed useful artifacts, including message content that led to SSH-related material. 

This was an important lesson from the lab: mail services are not only authentication targets, they are often repositories of sensitive operational data. Even when the services themselves are not vulnerable, the content they hold can be highly valuable.

## SNMP Enumeration

SNMP turned out to be one of the highest-value services on the target. After identifying the correct community string, SNMP enumeration exposed both system context and credential-related information. It provided confirmation of the host identity, operating system details, administrative contact information, and even references to user-specific operational artifacts.

More importantly, SNMP helped validate the credential path and supported the transition from external enumeration into authenticated access. This lab strongly reinforced how powerful SNMP can be when it is configured loosely.

## SSH Access and Local Enumeration

With valid credentials and SSH access established, the focus shifted to local enumeration. At this point, there was no immediate privilege escalation path through sudo, so the correct move was to inspect the user environment, mailbox storage, shell history, SSH material, and local artifacts that might reveal how the system was being used operationally.

Several useful findings came from this phase:

- Mail storage and Dovecot-related artifacts
- SSH key material associated with the current user
- Shell history showing prior database access
- User-specific files that hinted at domain and service usage
- Evidence that the current credentials were reused locally for other services 

This was not a lab where privilege came from a kernel issue or a sudo misconfiguration. Instead, it came from understanding how the user interacted with the system.

## Database Discovery

One of the most important local findings was evidence of MySQL usage. Shell history and local artifacts pointed toward database access, and validating that route led to a database containing user credential information. Once the relevant database was identified, the target account entry could be located and the required proof credential recovered.

This made the overall chain very clean:

1. External service enumeration
2. Mail protocol analysis
3. SNMP credential and context discovery
4. SSH access with validated credentials
5. Local artifact review
6. MySQL access and recovery of the target account data 

## Why this lab was interesting

This lab stands out because it rewards real operational thinking. There is no need to rush into exploitation when the environment is already leaking valuable information through normal services. Instead, the challenge is to notice what matters and connect it properly.

The most useful concepts practiced here were:

- Service fingerprinting across TCP and UDP
- Manual mail protocol interaction
- SNMP enumeration for context and credentials
- Credential reuse validation
- User-level local enumeration
- Database-backed credential recovery

## Lessons Learned

- Do not underestimate SNMP during footprinting; it can reveal both system context and credential material
- Mail services can expose operational secrets even when the protocol itself is not exploitable
- Valid credentials should always be tested across all relevant services
- After SSH access, shell history and user artifacts can be more valuable than privilege escalation checks
- Database traces in user history are often worth following immediately
- Good footprinting often solves the lab before exploitation is even necessary

## Tools Used

- Nmap
- Telnet / Netcat
- OpenSSL client
- onesixtyone
- snmpwalk
- SSH
- MySQL client
- Standard Linux enumeration commands

## Key Takeaways

Footprinting Lab - Hard is a strong example of how layered enumeration wins. The path was not about breaking one service, but about collecting small pieces of trustworthy information from several services and using them together.

The biggest lesson from this lab is that good operators do not just scan ports — they build context, validate assumptions, and follow the data wherever it leads.

## Artifacts Observed

- Linux mail server exposing POP3, IMAP, and SSH
- SNMP service disclosing operational context
- Mailbox content leading to SSH-related access
- Reused credentials across multiple services
- User shell history indicating MySQL interaction
- Database records containing the target account credentials
