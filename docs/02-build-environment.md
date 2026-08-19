# 02 — Environment Build

## Hypervisor

VirtualBox was used to host all four VMs on a single physical machine.

Minimum comfortable spec for running all four VMs concurrently:
- 16GB+ RAM (each Windows VM given 2–4GB, Splunk VM given more since it's
  doing continuous ingestion/search)
- 250GB+ free disk space
- A NAT Network created in VirtualBox (**Tools → Network → NAT Networks →
  Create**) with the IPv4 prefix `192.168.10.0/24` and DHCP left enabled at
  the network level (individual VMs still assigned static IPs inside the
  guest OS)

## VMs built

| VM | Base image | Purpose |
|---|---|---|
| `AD-DC01` | Windows Server 2022 (Standard, Desktop Experience) | Domain controller |
| `TARGET-PC` | Windows 10 | Domain-joined "victim" endpoint |
| `SPLUNK-SRV` | Ubuntu Server 22.04 | Splunk Enterprise host |
| `ATTACKER` | Kali Linux (official pre-built VirtualBox image) | Attack platform |

Notes on the build:

- **Windows Server 2022**: installed with the Desktop Experience option
  (not Server Core) — the CLI-only install is harder to navigate for a lab
  where the goal is learning AD administration hands-on, not minimising
  attack surface.
- **Windows 10**: standard custom install, renamed to `TARGET-PC` early on
  so it would be identifiable in Splunk/Event Viewer later, before any
  domain join.
- **Ubuntu Server**: installed without a desktop environment — Splunk is
  administered through its web UI (`:8000`) so a GUI on the host isn't
  needed. VirtualBox Guest Additions installed afterwards to enable shared
  folders for transferring the Splunk installer package into the VM.
- **Kali Linux**: used the official pre-built `.vbox` image rather than a
  manual ISO install, since Kali ships with its tooling pre-configured.

Every network adapter across all four VMs was switched from the VirtualBox
default (NAT) to the custom **NAT Network** created above, so they could
all see each other.

## Snapshot discipline

A snapshot was taken after each of the following milestones, since any one
of them is expensive to redo from scratch:
- Fresh OS install + updates, before any lab-specific configuration
- AD DS installed and DC promoted
- Splunk + Sysmon fully configured and confirmed ingesting
- Immediately before running the brute-force attack (so the "before" state
  is always recoverable for a re-run)

This meant that when something broke mid-way through a later stage, I could
revert to the last known-good snapshot instead of rebuilding the whole lab.
