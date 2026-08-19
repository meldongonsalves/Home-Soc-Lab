# 05 — Brute-Force Attack & Detection

With telemetry flowing, the next step was to generate a real attack against
the domain and then find it in Splunk — going through the same motions as a
SOC analyst triaging an alert.

> **Scope note:** this attack was run entirely against VMs I own, on an
> isolated lab network with no route to production systems. Brute-force
> tooling should never be pointed at infrastructure you don't have explicit
> permission to test.

## Attack setup

On `TARGET-PC`, Remote Desktop was enabled and the two domain users
(`j.smith`, `t.smith`) were added to the list of users permitted to connect
via RDP.

On `ATTACKER` (Kali Linux):

1. Installed **Crowbar**, a brute-forcing tool supporting RDP, SSH, VNC and
   OpenVPN.
2. Used the `rockyou.txt` wordlist bundled with Kali, trimmed to a small
   working subset for the demo, with the target account's actual password
   deliberately seeded into the list (this is a controlled lab exercise, not
   a blind brute-force — the point is to generate and study the resulting
   telemetry, not to test password strength).
3. Ran Crowbar against `t.smith` over RDP:

   ```
   crowbar -b rdp -u t.smith -C passwords.txt -s 192.168.10.100/32
   ```

4. Crowbar returned a successful RDP logon after working through the
   password list.

## Detecting it in Splunk

Switched to Splunk and searched the `endpoint` index, filtered to the last
15 minutes and the targeted username:

```
index=endpoint "t.smith"
```

Findings:

- **~20 events with Event ID `4625`** ("An account failed to log on") in a
  very tight time window — a strong signal of automated brute-forcing
  rather than a person mistyping a password a couple of times.
- **One event with Event ID `4624`** ("An account was successfully logged
  on") immediately following the run of failures — the successful
  compromise.
- Expanding the `4624` event showed the **source workstation name and IP
  matching the Kali attack box**, tying the successful logon directly back
  to the attacking host.

This is the same pattern a SOC alert for brute-force/credential-stuffing
activity is typically built on: a high rate of `4625` events from a single
source against a single account, followed by a `4624`. It maps to
**MITRE ATT&CK T1110 (Brute Force)**.

## Follow-up detection idea

A logical next step (not yet built in this lab) would be a saved Splunk
search/alert firing on "10+ Event ID 4625 for the same account within 1
minute," which is a common, low-noise way to catch this pattern in
production before the eventual successful logon happens.
