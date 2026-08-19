# 06 — Purple-Team Testing with Atomic Red Team

Detecting one brute-force attack proves the pipeline works, but it doesn't
say much about overall detection coverage. **Atomic Red Team** (a library of
small, scripted tests mapped to MITRE ATT&CK techniques) was used on
`TARGET-PC` to check what the lab could and couldn't see.

## Setup

- Ran PowerShell as Administrator and set `Set-ExecutionPolicy Bypass -Scope
  CurrentUser` to allow the test scripts to run.
- Added a Windows Defender exclusion for the whole `C:\` drive before
  installing — Defender otherwise flags and strips out several of Atomic
  Red Team's own test files as suspicious, which is expected (they
  intentionally resemble attacker behaviour) but breaks the framework if
  not excluded.
- Installed Atomic Red Team via its official PowerShell installer.

## Tests run

MITRE ATT&CK technique IDs were cross-referenced against the [ATT&CK
Enterprise Matrix](https://attack.mitre.org/matrices/enterprise/) to decide
what to test.

### T1136.001 — Create Account: Local Account

```
Invoke-AtomicTest T1136.001
```

Creates a new local user account (`Invoke-AtomicTest` names it something
like `new_local_user`). Searching Splunk immediately afterwards for
`index=endpoint "new local user"` returned **zero results**.

That's a genuine finding, not a failure of the exercise — Atomic Red Team's
value is exactly this: it surfaces **visibility gaps**. In this case, the
inputs.conf configuration on the forwarder wasn't yet capturing the
specific Security event ID for local account creation (Event ID `4720`),
so this technique would go completely undetected in the current setup — a
real gap worth fixing before relying on this lab as a genuine detection
baseline.

### T1059.001 — Command and Scripting Interpreter: PowerShell

```
Invoke-AtomicTest T1059.001
```

Windows Defender flagged suspicious PowerShell activity in real time
(`-exec bypass -noprofile`-style invocation). After waiting a short period
for indexing, the same search in Splunk **did** surface the event — showing
up with the matching command-line flags — confirming this technique class
*is* visible with the current Sysmon/Splunk configuration.

## Takeaways

- Detection coverage isn't uniform just because a SIEM is receiving logs —
  it depends entirely on which event IDs are actually being forwarded and
  indexed.
- T1136.001 (local account creation) is a concrete, documented gap in this
  lab's current `inputs.conf` — the fix is adding Event ID `4720` (and
  related account-management events) to the forwarded event set.
- Purple-teaming like this — attack, then check the SIEM, then fix the gap
  — is a fast, cheap way to validate detections before trusting them in
  production.

## Snapshot

The lab was snapshotted immediately after this stage so the "as-tested"
state (including the known 4720 gap) is preserved and can be revisited
after the fix is applied.
