# 03 — Active Directory Setup

## Promoting the domain controller

On `AD-DC01`:

1. Set a static IP (`192.168.10.7/24`, gateway `192.168.10.1`, DNS
   `8.8.8.8` initially — repointed at itself once AD DNS was live).
2. **Server Manager → Add roles and features → Active Directory Domain
   Services (AD DS)**.
3. Once AD DS installed, used the post-deployment notification to
   **promote this server to a domain controller**, choosing **Add a new
   forest** with domain name `homelab.local`.
4. Set the Directory Services Restore Mode (DSRM) password and let the
   wizard install DNS and finish the forest/domain creation. The server
   rebooted automatically to complete promotion.

Worth calling out: the domain's database file, `ntds.dit`, is one of the
highest-value targets for a real attacker — it holds every AD object
including password hashes. Any unauthorised access to that file on a real
domain controller should be treated as a full domain compromise.

## Creating structure: OUs and users

Rather than dropping users straight into the default `Users` container
(which doesn't reflect how a real organisation is laid out), I created
Organizational Units to mirror a basic departmental structure:

- `IT` OU → user `j.smith` (Jenny Smith)
- `HR` OU → user `t.smith` (Terry Smith)

Both created via **Active Directory Users and Computers → right-click OU →
New → User**, with "user must change password at next logon" unchecked
since this is a lab, not production.

## Domain-joining the target machine

On `TARGET-PC`:

1. Repointed the machine's DNS server to `192.168.10.7` (the DC) — without
   this, domain join fails because the machine can't resolve
   `homelab.local` at all.
2. **System Properties → Change → Member of: Domain** → entered
   `homelab.local`.
3. Authenticated with the DC's local administrator credentials (in a real
   environment this would instead be a dedicated, least-privilege service
   account authorised to join computers to the domain — using the built-in
   administrator account is a lab-only shortcut).
4. Rebooted, then logged in as `j.smith` using the domain sign-in option to
   confirm the join had worked end-to-end.

At this point the lab had a working single-domain AD environment: one DC,
two OUs, two users, and one domain-joined endpoint — the minimum viable
setup needed to generate realistic authentication telemetry later.
