# Home SOC Lab — Active Directory Attack Detection

A self-built, self-hosted Security Operations Centre (SOC) lab used to practise
attacking and defending a small Windows domain environment. The lab covers
the full loop: standing up an Active Directory domain, instrumenting it for
visibility, launching a real attack against it, and then detecting and
investigating that attack in a SIEM — the same workflow a junior SOC
analyst or IT security support engineer would follow in production.

All infrastructure is virtualised locally in VirtualBox. No cloud resources
or third-party infrastructure are used.

## Why this project

I built this to get hands-on with the tools and workflows that come up
constantly in IT support and SOC analyst job descriptions — Active Directory
administration, Sysmon, Splunk, and MITRE ATT&CK-mapped detection — rather
than only knowing them from theory. Everything here was built, broken, and
rebuilt by me on my own hardware.

## Architecture

| Host | Role | OS |
|---|---|---|
| `AD-DC01` | Domain Controller | Windows Server 2022 |
| `TARGET-PC` | Domain-joined endpoint | Windows 10 |
| `SPLUNK-SRV` | SIEM / log aggregation | Ubuntu Server 22.04 + Splunk Enterprise |
| `ATTACKER` | Attack platform | Kali Linux |

All four machines sit on an isolated VirtualBox NAT Network so they can
reach each other and the internet (for updates) without being exposed to my
home network. See [`docs/01-architecture.md`](docs/01-architecture.md) for
the full network diagram and IP scheme.

## What's documented here

1. [**Architecture & network design**](docs/01-architecture.md) — topology,
   IP addressing, and the reasoning behind the layout.
2. [**Environment build**](docs/02-build-environment.md) — installing and
   configuring all four VMs in VirtualBox.
3. [**Active Directory setup**](docs/03-active-directory-setup.md) —
   promoting the domain controller, creating OUs/users, and domain-joining
   the target machine.
4. [**Splunk & Sysmon deployment**](docs/04-splunk-sysmon-deployment.md) —
   getting endpoint telemetry flowing into a working SIEM.
5. [**Brute-force attack & detection**](docs/05-detection-bruteforce.md) —
   simulating an RDP brute-force attack and investigating it in Splunk
   using Windows Event IDs 4625/4624.
6. [**Purple-team testing with Atomic Red Team**](docs/06-purple-team-atomic-red-team.md) —
   running MITRE ATT&CK-mapped techniques to check detection coverage and
   surface visibility gaps.

## Skills demonstrated

- Active Directory Domain Services: forest/domain creation, OUs, users, domain join
- Windows networking: static IPs, DNS resolution, NAT networking in VirtualBox
- SIEM administration: Splunk Enterprise install, index configuration, receiving/forwarding
- Endpoint telemetry: Sysmon deployment and configuration, Splunk Universal Forwarder
- Attack simulation: RDP brute-forcing with Crowbar
- Log analysis: correlating Windows Security Event IDs to reconstruct an attack timeline
- Purple teaming: Atomic Red Team, MITRE ATT&CK technique mapping, detection gap analysis
- Documentation: snapshotting, note-taking and structured write-ups for repeatability

## Screenshots

See [`screenshots/`](screenshots) — network diagram, Splunk searches, and
event detail captures for each stage of the build.

## Related work

A second, separate home lab covering OT/SCADA protocol analysis, active
reconnaissance, DoS testing and phishing/social-engineering simulation is
documented in my [other cybersecurity home lab repo] (link to be added).

## Disclaimer

Everything in this lab runs entirely on infrastructure I own and control.
No attacks were run against any system I do not own or have explicit
permission to test.
