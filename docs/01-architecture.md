# 01 — Architecture & Network Design

Before installing anything, I mapped out the lab logically. Sketching the
topology out first — even roughly — makes it much easier to reason about
where traffic should and shouldn't flow, and it's exactly the kind of
diagram that comes up in interviews when you're asked to whiteboard a
network.

## Topology

```
                        ┌────────────────────┐
                        │      Internet       │
                        └──────────┬───────────┘
                                   │
                             ┌─────┴─────┐
                             │  Router    │
                             └─────┬─────┘
                                   │
                             ┌─────┴─────┐
                             │  L2 Switch │
                             └──┬───┬───┬─┘
                 ┌──────────────┘   │   └──────────────┐
                 │                  │                  │
        ┌────────┴────────┐ ┌──────┴───────┐ ┌────────┴────────┐
        │   AD-DC01        │ │  SPLUNK-SRV   │ │   TARGET-PC     │
        │ Windows Server   │ │ Ubuntu Server │ │  Windows 10     │
        │ 2022             │ │ + Splunk Ent. │ │ (domain-joined) │
        └──────────────────┘ └───────────────┘ └─────────────────┘
                                                          │
                                                 ┌────────┴────────┐
                                                 │   ATTACKER       │
                                                 │  Kali Linux      │
                                                 └──────────────────┘
```

All four VMs sit on a single VirtualBox **NAT Network** (not the default
per-VM NAT adapter), so they can route to one another directly while still
reaching the internet for updates and package installs. A NAT Network was
chosen over a Host-only network specifically because every VM needed
outbound internet access for updates and tool installation.

## IP addressing

| Host | Role | IP | Notes |
|---|---|---|---|
| `AD-DC01` | Domain Controller | `192.168.10.7/24` | Static |
| `SPLUNK-SRV` | Splunk Enterprise | `192.168.10.10/24` | Static, Splunk Web on :8000 |
| `TARGET-PC` | Domain-joined endpoint | `192.168.10.100/24` | Static, moved off DHCP to avoid clashing with the DC |
| `ATTACKER` | Kali Linux | `192.168.10.250/24` | Static |
| Gateway | NAT Network gateway | `192.168.10.1` | |
| DNS | Points at `AD-DC01` once the domain exists | `192.168.10.7` | Google DNS (`8.8.8.8`) used before AD DNS was live |

Domain name: `homelab.local` (an internal-only test domain — never used
against a real/public domain).

## Design decisions worth noting

- **Static IPs everywhere** — DHCP is fine for a single machine, but once
  Sysmon/Splunk forwarding and DNS resolution to a domain controller are in
  play, a machine's IP changing mid-lab breaks everything downstream. Static
  addressing across the board removed an entire class of "why did this stop
  working" troubleshooting.
- **Splunk on its own dedicated Ubuntu Server VM**, sized more generously
  (more RAM/CPU/disk) than the other VMs, since it's the one host doing
  continuous ingestion and search workloads.
- **Kali kept off the domain entirely** and not configured to forward logs
  to Splunk — it represents an external attacker, so nothing about its
  activity should be "trusted" telemetry the way endpoint logs are.
- **Snapshots taken at every stable milestone** (clean install, AD promoted,
  Splunk configured, etc.) so any stage could be broken and reverted without
  rebuilding from scratch.
