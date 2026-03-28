# MonitorsFour

- **Platform:** Hack The Box
- **Status:** Retired
- **OS:** Linux
- **Category:** Web / Containers
- **Focus Areas:** Web Enumeration, Virtual Host Discovery, Credential Exposure, Cacti, Container Pivoting, Docker Misconfiguration, Privilege Escalation

## Summary

MonitorsFour is a Linux machine that combines web enumeration, hidden host discovery, exposed application data, and container-to-host privilege escalation. The box starts with a small exposed surface, but deeper enumeration reveals additional application components and a much more interesting internal attack path.

What makes this machine stand out is the layered compromise. The initial foothold comes from careful web enumeration and understanding the application's behavior, while the final escalation depends on identifying the execution context and abusing infrastructure weaknesses rather than stopping at the first shell.

## Enumeration

An initial scan exposed a very limited external attack surface, with HTTP and a WinRM-related service visible from the outside. Early checks against the web application showed that the site was running PHP behind nginx, and further web fuzzing quickly revealed multiple interesting paths, including an exposed environment file and several application endpoints.

The exposed `.env` file provided backend configuration details, including the database hostname, port, and application credentials, which was the first strong sign that the target suffered from serious web-layer misconfiguration. Error messages on other routes also leaked internal filesystem paths and helped confirm how the application was structured on disk.

### Notable findings

- HTTP service exposed on port 80
- Additional Microsoft HTTP API service visible externally
- PHP application behind nginx
- Exposed `.env` file containing backend configuration
- Application error messages leaking local path information
- User-related endpoint behaving differently when supplied with a token parameter
- Hidden virtual host later discovered during host header fuzzing 

## Virtual Host Discovery

At first glance, the main site looked fairly simple and did not immediately expose a direct administrative weakness. The breakthrough came from broadening enumeration beyond standard directory discovery and checking for additional virtual hosts. That process uncovered a separate subdomain tied to a Cacti deployment, which significantly expanded the scope of the target and changed the entire direction of the assessment. :contentReference[oaicite:4]{index=4}

This was a key lesson from the box: when a web target feels too shallow, hidden hosts and alternate application surfaces may be where the real functionality lives.

## Credential Exposure and Application Access

Further inspection of the main application revealed a user-facing endpoint that returned internal user data when queried in a predictable way. The response exposed usernames, emails, roles, password hashes, and other employee-related information. Once the hash format was identified, one of the recovered values was cracked offline and reused successfully against application interfaces.

That access opened the way into the administrative portion of the web application and also enabled authentication against the Cacti instance discovered earlier. From an attacker’s perspective, this was a classic example of how a low-visibility information disclosure issue can become the central foothold when combined with weak credential handling.

### Key weaknesses chained together

- Sensitive configuration exposure
- Overly verbose application behavior
- Insecure user data disclosure
- Recoverable password hashes
- Credential reuse across services
## Initial Foothold

With valid access to the Cacti application, the next step was to move from authenticated web access to code execution. After validating that the deployment matched a known vulnerable condition, the application was abused to gain an interactive shell on the target environment. The resulting access confirmed that the compromise had landed inside a containerized context rather than directly on the final host. 

This distinction mattered a lot. Getting a shell was important, but it was not yet the end goal. The real value came from recognizing that the foothold existed inside an isolated runtime and that additional enumeration was required to understand what infrastructure sat behind it.

## Post-Exploitation

Once inside the compromised environment, local enumeration showed signs of containerization. Resolver configuration and local networking details pointed toward internal infrastructure that was not directly exposed externally. Internal service discovery then revealed multiple services reachable from the container, including one associated with Docker management. 

At this point, the challenge shifted away from the web application itself and into container escape / infrastructure abuse territory. This was the main turning point of the machine.

### Important observations after foothold

- Shell obtained in a constrained application environment
- Evidence that the foothold was inside a container
- Internal address space reachable from that context
- Additional services exposed only internally
- Misconfigured Docker-related interface accessible without proper restriction 

## Privilege Escalation

Privilege escalation relied on abusing exposed container management functionality from inside the compromised runtime. Because the internal Docker interface was insufficiently protected, it became possible to interact with the host’s container subsystem and reach the underlying operating system rather than remaining isolated inside the application container. 

This was the most important defensive takeaway from the box. Containerization by itself is not a security boundary if orchestration or management interfaces are exposed carelessly. In MonitorsFour, the web foothold was only the first stage; total compromise came from weak separation between application infrastructure and host control.

## Why this box was interesting

MonitorsFour rewards persistence more than speed. The machine starts with a small footprint, but every layer of enumeration uncovers another one:

- Standard web recon
- Configuration leakage
- Hidden host discovery
- Credential recovery
- Monitoring platform access
- Code execution in a container
- Internal infrastructure discovery
- Container-to-host escalation

Because of that, it feels less like a single exploit and more like a full compromise chain.

## Lessons Learned

- A limited initial attack surface can still hide a complex environment
- Exposed environment files are often much more valuable than they first appear
- Error messages and verbose responses can provide strong architectural clues
- Virtual host enumeration can completely change the assessment path
- After getting a shell, identifying whether you are on the host or in a container is critical
- Internal management services can be far more dangerous than the original web vulnerability

## Tools Used

- Nmap
- Nikto
- Gobuster / FFUF
- Burp Suite
- Hashcat
- Curl
- Linux enumeration utilities

## Key Takeaways

MonitorsFour is a strong example of a layered attack path. The initial foothold depended on careful web enumeration and exposed application data, but the final success required understanding the execution context and pivoting into infrastructure weaknesses.

The machine reinforces an important idea: post-exploitation awareness matters just as much as exploitation itself. Knowing where your shell lives, what it can reach, and how the surrounding environment is misconfigured is what turns partial access into full compromise.

## Artifacts Observed

- Main application exposing configuration data
- Hidden Cacti instance on a secondary virtual host
- User records exposed through an insecure application endpoint
- Offline-crackable password hashes
- Shell obtained in a containerized environment
- Internally reachable Docker-related service enabling host-level pivot
