# Challenge 13 (T0041) - Disastrous DNS Destruction

## Challenge Info
**Author:** Bailey Kasin<br>
**Framework Category:** Protect and Defend<br>
**Specialty Area:** Incident Response<br>
**Work Role:** Cyber Defense Incident Responder<br>
**Task Description:** Coordinate and provide expert technical support to enterprise-wide cyber defense technicians to resolve cyber defense incidents.

### Scenario
We are experiencing DNS related issues that were caused by a former employee's malicious activity. We need you to discover the root of the issue and restore access to the production website.

### Additional Information
More details and objectives about this challenge will be introduced during the challenge meeting, which will start once you begin deploying the challenge.

You will be able to check your progress during this challenge using the check panel within the workspace once the challenge is deployed. The checks within the check panel report on the state of some or all of the required tasks within the challenge.

Once you have completed the requested tasks, you will need to document the methodology you used with as much detail and professionalism as necessary. This should be done on the documentation tab within the workspace once the challenge is deployed. Below the main documentation section be sure to include a tagged list of applications you used to complete the challenge.

Your username/password to access all virtual machines and services within the workspace will be the following...<br>
Username: `playerone`<br>
Password: `password123`

The username/password used to access the Firewall's web interface within the workspace will be the following...<br>
Username: `admin`<br>
Password: `password123`

## Meeting Notes
![](../images/challenge13/meeting_notes.png)

## Network Map
![](../images/challenge13/PD-map.jpg)

## Documentation
As the challenge notes and meeting notes stated, users could not resolve `google.com` normally. Instead, the users are taken to a suspicious web page and are then prompted to download an executable file called `FastCleanup.exe`.

![](../images/challenge13/bad_dns.png)

To begin resolving this issue, I noted down the default DNS server that was being used in the network: `172.16.30.55`

This server was the `AD-Server`.

![](../images/challenge13/default_dns_server.png)

I logged into `AD-Server` and inspected its current DNS forwarders. This was to verify that the server was not forwarding public domain queries to a potentially malicious recursive lookup server.

However, the configuration seemed normal; the current forwarding server was `172.31.2.1`, and this was the ISP's gateway.

![](../images/challenge13/dns_forwarder.png)

To verify that the issue was rooted on `AD-Server` and no where else, I made a series of DNS queries for `google.com` to other reputable public recursive DNS services--and to `AD-Server` itself.

It seemed that `AD-Server` was polluted with an *A*-type DNS record that resolved `google.com` to an internal server: `172.16.30.100` (`Fileshare`).

![](../images/challenge13/dns_results.png)

To resolve the DNS poisoning issue, I cleared the DNS cache from `AD-Server`, and that did the trick.

![](../images/challenge13/clear_cache.png)

Next, I made a series of hardening changes to `AD-Server`:

1. I enabled DNSSEC validation for publicly resolved DNS queries.
2. I enabled DNS zone signing for the zone `prettysafeelectronics.io` (which was managed by `AD-Server`).
3. I enabled DNS cache pollution protection.

![](../images/challenge13/enable_dnssec.png)

![](../images/challenge13/dns_zone_signing.png)

![](../images/challenge13/pollution_protection.png)

The final task was to investigate the `Fileshare` server to find a sample of the malware artifact being hosted on it (i.e. `FastCleanup.exe`) and to triage and remove the website and original malware file.

![](../images/challenge13/malware.png)

![](../images/challenge13/rm.png)

## NICE Framework & CAE KU Mapping

### NICE Framework KSA
- K0001. Knowledge of computer networking concepts and protocols, and network security methodologies.
- K0002. Knowledge of risk management processes (e.g., methods for assessing and mitigating risk).
- K0004. Knowledge of cybersecurity and privacy principles.
- K0034. Knowledge of network services and protocols interactions that provide network communications.
- K0042. Knowledge of incident response and handling methodologies.
- K0167. Knowledge of system administration, network, and operating system hardening techniques.
- K0179. Knowledge of network security architecture concepts including topology, protocols, components, and principles (e.g., application of defense-in-depth).
- K0565. Knowledge of the common networking and routing protocols (e.g. TCP/IP), services (e.g., web, mail, DNS), and how they interact to provide network communications.
- S0077. Skill in securing network communications.

### CAE Knowledge Units
- Basic Cyber Operations
- Basic Networking
- Data Administration
- Digital Communications
- IT Systems Components
- Network Technology and Protocols